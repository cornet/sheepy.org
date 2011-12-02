--- 
layout: post
title: Rake for config deployment
---

I'm a great advocator of [puppet](http://puppetlabs.com/) or any configuration management tool. However sometimes it's an overkill. We have a number of servers which are not puppet controlled for various reason but we push new configurations out to them regularly.

For some things we have scripts to push the configs out and of course, some we don't. This occasionally leads to problems.

Taking the example of a DNS server, easy to configure, easy to mess up and easy not to notice when you've messed up (until the TTL runs out).

The more commands you have to issue, the more likely a mistake is to be made.

There are many hammers for this nail but ruby is my weapon of choice these days so I've knocked up a quick Rakefile to do everything. So all I need do is
    rake deploy
and it'll do the following:
* Sync the config out
* Set the correct permissions
* Run named-checkzone against all the zonefiles
* Reload the config if the checks pass

To avoid creating loads of SSH connections I've employed the [Net::SSH::Multi](https://github.com/jamis/net-ssh-multi) gem which is definitly worth a look.

{% highlight ruby %}
#
# Configuration
#
$ssh_user = "foo"
$ssh_host = "example.com"
remote_root = "/etc/bind"


rsync = 'rsync -a -e "ssh" --rsync-path="sudo rsync"'
remote_dir = "#{$ssh_user}@#{$ssh_host}:#{remote_root}/zones/"
local_dir  = "zones/"

def remote
  @remote ||= begin
    require 'net/ssh/multi'
    session = Net::SSH::Multi.start
    session.use "#{$ssh_user}@#{$ssh_host}"
    session
  end
end


desc "Get config from #{$ssh_host}"
task :get do
  puts "*** Getting config from #{$ssh_host} ***"
  system "#{rsync} #{remote_dir} #{local_dir}"
end

desc "Put config to #{$ssh_host}"
task :put do
  puts "*** Deploying config files to #{$ssh_host} ***"
  system "#{rsync} #{local_dir} #{remote_dir}"
end

desc "Test config on #{$ssh_host}"
task :test do
  puts "*** Testing zone files ***"

  errors = Array.new 

  FileList.new("zones/*").each do |zone|
    output = Array.new
    channel = remote.exec("sudo named-checkzone #{zone.sub(/zones\/db./, '')} #{remote_root}/#{zone}") do |ch, stream, data|
      output << "[#{ch[:host]} : #{stream}] #{data}"
    end

    channel.wait

    if channel.any? { |c| c[:exit_status] != 0 }
      errors << "#{zone} failed to validate"
      errors << output.join
    end

  end

  unless errors.empty?
    abort "#{errors.join("\n")}"
  end
end

desc "Set correct permissions"
task :set_perms do
  puts "*** Setting correct file permissions ***"
  remote.exec("sudo chmod 640 /etc/bind/zones/db.*").wait
  remote.exec("sudo chown bind:bind /etc/bind/zones/db.*").wait
end

desc "Reload bind on #{$ssh_host}"
task :reload do
  puts "*** Reloading Bind ***"

  channel = remote.exec("sudo rndc reload") do |ch, stream, data|
    puts "[#{ch[:host]} : #{stream}] #{data}"
  end

  channel.wait
end

desc "Put config, test and reload"
task :deploy => [:put, :set_perms, :test, :reload] do
  puts "*** Zonefiles deployed ***"
end
{% endhighlight %}


Yes it's hacky and could do with some improvement but then I'm a sysadmin and not a developer. I might have a go at writting a gem to make these sort of Rakefiles cleaner but it's not too bad.
