task :default => :dev

desc 'Ping pingomatic'
task :pingomatic do
  begin
    require 'xmlrpc/client'
    puts '* Pinging ping-o-matic'
    XMLRPC::Client.new('rpc.pingomatic.com', '/').call('weblogUpdates.extendedPing', 'Swedish Pixels' , 'http://swedishpixels.com', 'http://feeds.feedburner.com/swedishpixels/masterfeed')
  rescue LoadError
    puts '! Could not ping ping-o-matic, because XMLRPC::Client could not be found.'
  end
end

desc 'Notify Google of the new sitemap'
task :sitemap do
  begin
    require 'net/http'
    require 'uri'
    puts '* Pinging Google about our sitemap'
    Net::HTTP.get('www.google.com', '/webmasters/tools/ping?sitemap=' + URI.escape('http://swedishpixels.com/sitemap.xml'))
  rescue LoadError
    puts '! Could not ping Google about our sitemap, because Net::HTTP or URI could not be found.'
  end
end

desc 'Run Jekyll in development mode'
task :dev do
  puts '* Running Jekyll with auto-generation and server'
  puts `jekyll --auto --server`
end

desc 'Run Jekyll to generate the site'
task :build do
  puts '* Generating static site with Jekyll'
  puts `jekyll`
end

desc 'rsync the contents of ./_site to the server'
task :sync do
  puts '* Publishing files to live server'
  puts `rsync -avz "_site/" hertze@hertze.com:~/public_html/swedishpixels %w(-ave 'ssh -p 22123')`
end

desc 'Push source code to Github'
task :push do
  puts '* Pushing to Github'
  puts `git push git@github.com:hertze/hertze.github.com.git master`
end

desc 'Create and push a tag'
task :tag do
  t = ENV['T']
  m = ENV['M']
  unless t && m
    puts "USAGE: rake tag T='1.0-my-tag-name' M='My description of this tag'"
    exit(1)
  end

  puts '* Creating tag'
  puts `git tag -a -m "#{m}" #{t}`

  puts '* Pushing tags'
  puts `git push github master --tags`
end

desc 'Generate and publish the entire site, and send out pings'
task :publish => [:push, :sitemap, :pingomatic] do
end

desc 'Generate and publish the entire site, and send out pings'
task :ping => [:sitemap, :pingomatic] do
end

desc 'create a new draft post'
task :post do
  title, slug = get_title
  file = File.join(File.dirname(__FILE__), '_posts', slug + '.markdown')
  create_blank_post(file, title)
  open_in_editor(file)
end

desc 'List all draft posts'
task :drafts do
  puts `find ./_posts -type f -exec grep -H 'published: false' {} \\;`
end

# Helper method for :draft and :post, that required a TITLE environment
# variable to be set. If there is none, the task will exit.
#
# If there is a title given, then this method will return it and a escaped
# version suitable for filenames.
def get_title
  unless title = ENV['TITLE']
    puts "USAGE: rake post TITLE='the post title'"
    exit(1)
  end
  return [title, "#{Date.today}-#{title.downcase.gsub(/[^\w]+/, '-')}"]
end

# Helper method for :draft and :post, that will create a file at a given
# location and fill it with an empty post.
def create_blank_post(path, title)
  # Create the directories to this path if needed
  FileUtils.mkpath(File.dirname(path))

  # Write the template to the file
  File.open(path, "w") do |f|
    f << <<-EOS.gsub(/^    /, '')
    ---
    layout: post
    title: #{title}
    published: false
    categories:
    ---

    EOS
  end
end

# Helper method to open a file in the default text editor.
def open_in_editor(file)
  if (ENV['EDITOR'])
    system ("#{ENV['EDITOR']} #{file}")
  end
end