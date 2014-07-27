require 'json_watch'
require 'redis'
require 'redis/namespace'
require 'redis/pool'
require 'NIFTY'
require 'mail'

puts "start to watch -1"

Mail.defaults do
  delivery_method :smtp, {
    :address => 'smtp.sendgrid.net',
    :port => '587',
    :domain => 'heroku.com',
    :user_name => ENV['SENDGRID_USERNAME'],
    :password => ENV['SENDGRID_PASSWORD'],
    :authentication => :plain,
    :enable_starttls_auto => true
  }
end

class Watch < JsonWatch
	%w(east-1 west-1).each do |region|
		watch :"#{region} instances", notify: [:stdout, :mail], exclude: 'requestId' do
			compute(region).describe_instances.to_hash
		end
	end

	notify :stdout do |watch, diff|
		STDOUT.puts title(watch)
		STDOUT.puts JSON.pretty_generate(diff)
	end

	notify :mail do |watch, diff|
		subject = title(watch)
		Mail.deliver do
		  to ENV['TO_MAIL_ADDRESS']
		  from 'tily-watch@herokuapp.com'
		  subject subject
		  body JSON.pretty_generate(diff)
		end
	end

	class << self
		def title(watch)
			"#{watch[:name]} changed!"
		end

		def compute(region)
			api = NIFTY::Cloud::Base.new(
				access_key: ENV['ACCESS_KEY_ID'],
				secret_key: ENV['SECRET_ACCESS_KEY'],
				server: "#{region}.cp.cloud.nifty.com",
				path: '/api',
				socket_timeout: 300,
				connection_timeout: 300
			)
		end
	end
end

desc 'watch'
task :watch do
	puts "start to watch -1"
	redis = Redis::Namespace.new(:watch, redis: Redis::Pool.new(url: ENV['REDISTOGO_URL'] || 'redis://localhost:6379/15'))
	watch = Watch.new(cache: redis, sleep: 60*5)
	puts "start to watch"
	watch.start
end
