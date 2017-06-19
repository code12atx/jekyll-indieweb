require 'rake'
require 'yaml'
require 'rss'
require 'webmention'
require 'time'

def last_passing_timestamp(travis_url)
  ts = ''
  rss = RSS::Parser.parse(travis_url, false)
  rss.items.each do |item|
    if item.summary.to_s =~ /State: passed/
      ts = item.updated.content.to_s
      break
    end
  end
  ts
end

desc "Send webmentions"
task :webmention do
  project_owner = ENV['PROJECT_OWNER']
  project_name = ENV['PROJECT_NAME']
  rss_url = ENV['RSS_URL']

  puts "project owner: #{project_owner}"
  puts "project name: #{project_name}"
  puts "rss url: #{rss_url}"

  url = "https://api.travis-ci.org/repos/#{project_owner}/#{project_name}/builds.atom"
  latest = last_passing_timestamp(url) || 'Fri, 15 Jan 2016 23:59:59 +0000'

  puts "latest webmention time: #{latest}"

  last_time = Time.parse(latest)
  rss = RSS::Parser.parse(rss_url, false)
  rss.items.each do |item|
    t = Time.parse(item.pubDate.to_s)
    # XXX assumes new blog post was not created prior to last passing travis build
    if t > last_time
      puts "sending webmentions for #{item.link}"
      client = Webmention::Client.new item.link
      sent = client.send_mentions
    end
  end

end
