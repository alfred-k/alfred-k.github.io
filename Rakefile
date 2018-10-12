POST_DIR = './_posts'
POST_TEMPLATE = '_template.md'

task :default => :build

desc 'Clean up generated site'
task :clean do
  cleanup
end

desc 'Test site output for Liquid template errors'
task :test do
  errors = `grep --exclude Rakefile -R 'Liquid error:' _site`
  if errors.nil? || errors.empty?
    puts "No errors"
  else
    puts "Errors:"
    puts errors.inspect
    exit 1
  end
end

namespace :build do
  desc 'Build site with Jekyll'
  task :development => :clean do
    jekyll 'development'  #('--lsi')
  end
  task :production => :clean do
    jekyll 'production'
  end
end

task :build => 'build:development'

desc 'Start server with --auto'
task :server => :clean do
  jekyll 'development', '--server --auto'
end

desc 'Build and deploy'
task :deploy => ['build:production', :test] do
  deploy
end

desc 'Make a new post'
task :post, [:name] do |t, args|
  if !FileTest.directory?(POST_DIR)
    abort("rake aborted: '#{POST_DIR}' directory not found.")
  end

  # Get post template
  template_file = File.join(POST_DIR, POST_TEMPLATE)
  if !File.exists?(template_file)
    abort("rake aborted: post template '#{template_file}' not found.")
  end
  template = File.open(template_file, "rb") do |f|
    f.read
  end

  # Variables
  title = args.name || ENV['title'] || 'new-post'
  tags = (ENV['tags'] || '').split(',')
  slug = slugize(title)
  begin
    puts ENV['date']
    time = ENV['date'] ? Time.new(ENV['date']) : Time.now
    date = time.strftime('%Y-%m-%d')
  rescue Exception => e
    puts 'Error: date format must be YYYY-MM-DD'
    exit -1
  end

  # Render template
  contents = template.
    gsub('%title%', title).
    gsub('%date%', time.strftime("%Y-%m-%d %H:%M:%S %z")).
    gsub('%tags%', "[ #{tags.join(', ')} ]")

  filename = File.join(POST_DIR, "#{date}-#{slug}.md")
  if File.exist?(filename)
    abort('rake aborted!') if ask("#{filename} already exists. Do you want to overwrite?", ['y', 'n']) == 'n'
  end
  
  puts "Creating new post: #{filename}"
  File.open(filename, 'wb') do |f|
    f.write contents
  end
end

def cleanup
  sh 'rm -rf _site'
end

def jekyll(env = 'development', opts = '')
  ENV['JEKYLL_ENV'] = env
  sh "jekyll #{opts}"
end

def deploy
  sh 'jekyll-s3 -h'
end

# TODO: accept unicode slugs
# see: https://github.com/mozilla/unicode-slugify
def slugize(text)
  text.downcase.strip.gsub(' ', '-').gsub(/[^\w-]/, '')
end

def ask(message, valid_options)
  if valid_options
    answer = get_stdin("#{message} #{valid_options.to_s.gsub(/"/, '').gsub(/, /,'/')} ") while !valid_options.include?(answer)
  else
    answer = get_stdin(message)
  end
  answer
end

def get_stdin(message)
  print message
  STDIN.gets.chomp
end
