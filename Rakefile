# encoding: UTF-8

require "yaml"
require "nokogiri"
require "open-uri"

def extract_titles_from_html(html)
  html.css("a[name] > b").map(&:text).map(&:strip)
end

def clean_number_string(number_string, position=:start)
  replacement_re = /[x`]/ # placeholder x or backtick
  numbers_re = /[^\d^x^`]/ # not digits, x, or backtick

  number_string.gsub!(numbers_re, "")

  if position == :start
    number_string.gsub!(replacement_re, "0")
  else
    number_string.gsub!(replacement_re, "9")
  end
  number_string
end

def extract_tables_from_html(html, table_offset=0)
  tables = html.css("table")

  # Headers
  # Type, Lens, Country, Scr, Features, Start No, Confirmed, End No, Qty, Date
  # headers = tables[0].css("th").map(&:text).map(&:strip)
  headers = [:type, :lens, :country, :scr, :features, :start_no, :confirmed, :end_no, :qty, :date]

  tables = tables.map do |table|
    output = []

    table.css("tr").each do |tr|
      row = tr.css("td").map(&:text).map(&:strip)
      output << Hash[headers.zip(row)] unless row.empty?
    end

    output
  end

  tables = tables.drop(table_offset)

  tables.each do |lenses|
    lenses.each do |lens|
      range_re = /([\d`]+)[\.\s-]+([\d`]+)/
      replacement_re = /[x`]/

      start_no = lens[:start_no]
      end_no = lens[:end_no]
      confirmed = lens[:confirmed]

      numbers_re = /[^\d^x^`]/


      start_no = clean_number_string(start_no) if start_no
      start_no = start_no.to_i

      end_no = clean_number_string(end_no) if end_no
      end_no = end_no.to_i

      confirmed.gsub!(/[^\d^\-^\s]/, "")

      range = if start_no > 0 && end_no > 0
        (start_no)..(end_no)
      elsif start_no > 0 && confirmed
        if (matches = confirmed.match(range_re))
          (start_no)..(matches[2].gsub(/[x`]/, "9").to_i)
        elsif confirmed =~ /^\d+$/
          confirmed.to_i..confirmed.to_i
        else
          start_no..start_no
        end
      elsif start_no == 0 && end_no == 0 && confirmed
        if (matches = confirmed.match(range_re))
          (matches[1].gsub(/[x`]/, "0").to_i)..(matches[2].gsub(/[x`]/, "9").to_i)
        elsif confirmed =~ /^\d+$/
          confirmed.to_i..confirmed.to_i
        else
          nil
        end
      elsif end_no > 0 &&  start_no == 0
        end_no..end_no
      else
        nil
      end

      lens[:range] = range
    end

  end
  tables
end

def map_titles_and_tables(titles, tables)
  hash = {}

  titles.each_with_index do |title, index|
    hash[title] = tables[index]
  end

  hash
end

namespace :nikon do
  desc "Delete the old reports"
  task :serials do
    html = Nokogiri::HTML(open("http://www.photosynthesis.co.nz/nikon/serialno.html"))

    titles = extract_titles_from_html(html)
    tables = extract_tables_from_html(html, 2)

    hash = map_titles_and_tables(titles, tables)

    puts YAML.dump hash
  end
end

task default: ["nikon:serials"]