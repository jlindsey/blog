namespace :site do
  desc "Cleans all generated files"
  task :clean do
    require 'fileutils'
    puts "Cleaning generated files"
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
end

task :default => ["site:clean", "site:deploy"]

namespace :post do
  desc "Generates draft for a new post"
  task :new do
    require 'fileutils'
    FileUtils.mkdir 'drafts' unless File.exists? 'drafts'

    puts "Post title?"
    title = STDIN.gets.strip
    safe_title = title.downcase.gsub(' ', '-').gsub(/[^\w-]/, '')

    file_path = File.join('drafts', "#{safe_title}.markdown")

    File.open file_path, 'w' do |f|
      f.puts <<-EOS
---
layout: post
date: %DATE%
time: %TIME%
title: #{title}
---

      EOS
    end

    puts "\nGenerated #{file_path}"

    puts "Open now? [Y/n]"
    open = STDIN.gets.strip.upcase
    exec %Q(#{ENV['EDITOR']} #{file_path}) if open == 'Y' or open.empty?
  end

  desc "Publishes a draft post"
  task :publish do
    require 'fileutils'

    drafts = Dir['drafts/*.markdown']

    puts "Select a draft to publish:"
    drafts.each_with_index do |draft, i|
      title = File.read(draft).scan(/^title:\s?(.*)$/).pop.pop
      puts "#{i + 1}: #{title}"
    end

    loop do
      print "> "
      $n = STDIN.gets.strip.to_i

      if drafts.at($n-1).nil?
        puts "Option out of range. Pick again."
        next
      end

      break
    end

    draft = drafts.at($n-1)
    time_str = Time.now.strftime "%r"
    date_str = Time.now.strftime "%F"

    File.open("_posts/#{date_str}-#{File.basename(draft)}", 'w') do |f|
      f.puts File.read(draft).sub!('%DATE%', date_str).sub!('%TIME%', time_str)
    end

    FileUtils.rm_f draft
    puts "Published #{draft}"
  end
end

