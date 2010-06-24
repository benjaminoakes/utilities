#!/usr/bin/env ruby

# If you're doing this often from the command line, you can also add these to your ~/.sqliterc:
# 
#     .headers on
#     .mode csv
# 
# (That depends on external settings though -- this is meant to be a part of a script.)
# 
# Author: Benjamin Oakes <hello@benjaminoakes.com>

require 'csv'
require 'stringio'

require 'rubygems'
require 'sqlite3'

def show_usage
  STDERR.puts <<EOF
Usage: #{$PROGRAM_NAME} [--help] database_path [table_name_1 ... table_name_n] [< query.sql]"

Turns a SQLite3 database into a CSV file based on table names or a query.

  --help            Show this help text.
  database_path     Input SQLite3 database.
  table_names       Tables in the database.  Each will be output as a CSV file.
EOF
  exit(-1)
end

# Hacked together (and very simplistic) database interface.
class Database
  attr_accessor :query

  def initialize(database_path)
    @database = SQLite3::Database.new(File.expand_path(database_path))
  end

  def fields_for(table_name)
    if @query.nil?
      query = 'select * from ' + table_name
    else
      query = @query
    end

    statement = @database.prepare(query)

    return statement.columns
  end
  
  def select_all_from(table_name)
    rows = []

    if @query.nil?
      query = 'select * from ' + table_name
    else
      query = @query
    end

    puts query
    puts
  
    @database.execute(query) do |row|
      rows << row
    end
    
    return rows
  end
end

database_path = ARGV[0]
table_names = ARGV[1, ARGV.length]

if ARGV.any? { |a| '--help' == a } ||  database_path.nil?
  show_usage
end

database = Database.new(database_path)

puts "Reading query from STDIN:"
query = STDIN.readlines.join

unless query.empty?
  database.query = query
end

io = StringIO.new

table_names.each do |table_name|
  CSV::Writer.generate(io) do |csv|
    csv << database.fields_for(table_name)

    database.select_all_from(table_name).each do |row|
      row.each_with_index do |value, index|
        # # If you like (or require)  your boolean output to be different:
        # if 'f' == value
        #   row[index] = 'false' # 0
        # elsif 't' == value
        #   row[index] = 'true' # 1
        # else
        #   # Don't convert
        # end
      end
  
      csv << row
    end
  end
end

puts io.readlines.join

