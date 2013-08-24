# encoding: UTF-8

require "yaml"
require "nokogiri"
require "open-uri"

namespace :nikon do
  desc "Delete the old reports"
  task :serials do
    table_offset = 2

    html = Nokogiri::HTML(open("http://www.photosynthesis.co.nz/nikon/serialno.html"))

    titles = html.css("a[name] > b").map(&:text)
    tables = html.css("table")

    tables = tables.map do |table|
      output = []
      headers = table.css("th").map &:text
      output << headers

      table.css("tr").each do |tr|
        output << tr.css("td").map(&:text)
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