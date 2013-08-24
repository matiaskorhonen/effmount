# encoding: UTF-8

require "yaml"
require "nokogiri"
require "open-uri"

namespace :nikon do
  desc "Delete the old reports"
  task :serials do
    table_offset = 2

    html = Nokogiri::HTML(open("http://www.photosynthesis.co.nz/nikon/serialno.html"))

    titles = html.css("a[name] > b").map(&:text).map(&:strip)
    tables = html.css("table")

    tables = tables.map do |table|
      output = []

      # Headers
      # Type, Lens, Country, Scr, Features, Start No, Confirmed, End No, Qty, Date
      headers = table.css("th").map(&:text).map(&:strip)
      output << headers

      table.css("tr").each do |tr|
        output << tr.css("td").map(&:text).map(&:strip)
      end

      output
    end

    hash = {}

    titles.each_with_index do |title, index|
      hash[title] = tables[index + table_offset]
    end

    puts YAML.dump hash
  end
end

task default: ["nikon:serials"]