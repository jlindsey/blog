desc "Cleans all generated files"
task :clean do
  require 'fileutils'
  ['log', 'public'].each { |dir| FileUtils.rm_rf dir }
end

desc "Regenerate, commit, push, run chef"
task :deploy do
  puts "Regenerating site"
  %x(jekyll)
  puts "Committing files"
  %x(git add public/ && git commit -m "Regenerated for $(date)" && git push origin master 2>&1)
  puts "Running chef-client"
  %x(ssh -i ~/.ssh/jlindsey.pem ubuntu@www.joshualindsey.net "sudo chef-client")
end

namespace :post do
  desc "Generates a file for a new post"
  task :new do
    require 'fileutils'
    FileUtils.mkdir 'drafts' unless File.exists? 'drafts'

    puts "Post title?"
    title = STDIN.gets.strip
    safe_title = title.downcase.gsub(' ', '-').gsub(/[^\w-]/, '')

    time = Time.now
    time_str = time.strftime "%r"
    date_str = time.strftime "%F"

    file_path = File.join('drafts', "#{date_str}-#{safe_title}.markdown")

    File.open file_path, 'w' do |f|
      f.puts <<-EOS
---
layout: post
date: #{date_str}
time: #{time_str}
title: #{title}
---

      EOS
    end

    puts "\nGenerated #{file_path}"

    puts "Open now? [Y/n]"
    open = STDIN.gets.strip.upcase
    exec %Q(#{ENV['EDITOR']} #{file_path}) if open == 'Y' or open.empty?
  end
end

