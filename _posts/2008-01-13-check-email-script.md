--- 
layout: post
title: Check Email Script
created: 1200189871
---
So I've been playing with <a href="http://conky.sf.net/">conky</a> again and wanted it to check all my email accounts for new mail.

I found <a href="http://gentoo-wiki.com/TIP_conky#IMAP_Email_Checking_Script">this</a> script but I wanted to check multiple accounts and show the total mail count in my inbox.

After some hacking here is what I came up with... and yes I've removed the base64 encoding as it doesn't add any security what so ever!

    #!/usr/bin/env ruby
    
    require "optparse"
    require "net/imap"
    #
    # Parse options
    #
    def parse_options(args)
    
      # Hash to hold options
      options = Hash.new
    
      parser = OptionParser.new do |opts|
    
        opts.on('-H host', '--host host', 'Host') do |v|
          options[:host] = v
        end
    
        opts.on('-u user', '--user user', 'User') do |v|
          options[:user] = v
        end
    
        opts.on('-p password', '--password password', 'Password') do |v|
          options[:password] = v
        end
    
        opts.on('-h', '--help', 'Displays usage information') do
          puts opts
          exit 1
        end
      end
    
      # Parse Parameters
      begin
        parser.parse!(args)
      rescue OptionParser::ParseError => e
        puts "Parse Error: " + e
        puts parser.to_a
      end
    
      # Check for required params
      if (options.has_key?(:host) && options.has_key?(:user) && options.has_key?(:password))
        #
        # Return options
        #
        options
      else
        puts parser.to_a
        exit 1
      end
    
    end
    options = parse_options(ARGV)
    
    #
    # Lets check some mail!
    #
    imap = Net::IMAP.new(options[:host])
    imap.login(options[:user], options[:password])
    
    #
    # Get total mail
    #
    status = imap.status("inbox", ["MESSAGES", "UNSEEN"])
    puts "#{status['UNSEEN']}/#{status['MESSAGES']}"
    imap.disconnect
