desc "Creates a new blog post"
task :blog do
  print "Please enter in the title of the blog post: "
  title = $stdin.gets.chomp.strip
  name = title.gsub(/\s+/, '-')
  name = name.gsub(/[^a-zA-Z0-9_-]/, "").downcase
  time = Time.now.strftime("%Y-%m-%d")
  File.open("_posts/#{time}-#{name}.markdown", "w+") do |file|
    file.puts <<-EOF
--- 
title: #{title}
layout: post
tags:
---
EOF
  end
  puts "Created '_posts/#{time}-#{name}.markdown'"
end
