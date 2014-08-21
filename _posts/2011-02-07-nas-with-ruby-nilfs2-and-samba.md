---
layout: post
title: NAS with Ruby, NILFS2 and Samba
date: 2011-02-07 21:10
author: Giacomo Graziosi
comments: true
categories: [linux, sysadmin]
tags: [linux, nas, nilfs, nilfs2, open source, programming, ruby, samba]
---
Here comes a simple script I wrote to setup a couple of file servers using Linux and NILFS2 with daily incremental snapshots, sharing of the snapshots via Samba and optional sync on an external NTFS hard disk on USB: [nilfs2-ruby-nas](https://github.com/esistgut/nilfs2-ruby-nas).

As usual be extremely careful with this code as it did not receive proper testing and should be seen as a starting point for your own setups rather than a ready to use solution.

Take a look on the class I wrote to run NILFS2 commands, it isn't exactly fail proof :-D:

{% highlight ruby %}
class NILFS2
    def initialize(device)
        raise IOError, "can't find device file" unless File.exists?(device)
        @device = device
    end
    
    def get_checkpoints()
        t = `lscp #{@device}`
        r = Array.new
        t.split("\n")[1..-1].each do |l|
            a = l.split
            time = Time.parse("#{a[1]} #{a[2]}")
            r.push({:CNO => a[0], :MODE => a[3], :FLG => a[4],
            :NBLKINC => a[5], :ICNT => a[6], :time => time})
        end
        return r
    end

    def get_snapshots()
        r = Array.new
        get_checkpoints.each { |c| r.push(c) if c[:MODE] == "ss" }
        return r
    end

    def ss_to_mount(snapshot, path)
        return {:dev => @device, :path => path, :fs => "nilfs2", :opts => {"ro"=>nil, "cp" => snapshot[:CNO]} }
    end
    
    def make_checkpoint(snapshot=false)
        snapshot ? `mkcp -s #{@device}` : `mkcp #{@device}`
    end
    
    def make_snapshot()
        make_checkpoint(true)
    end
    
    def remove_checkpoint(checkpoint)
        `chcp cp #{@device} #{checkpoint[:CNO]}` if checkpoint[:MODE] == "ss"
        `rmcp #{@device} #{checkpoint[:CNO]}`
    end
    
    def get_total_space()
        `df -Pk #{@device} |grep ^/ | awk '{print $2;}'`.to_i * 1024
    end
    
    def get_used_space()
        `df -Pk #{@device} |grep ^/ | awk '{print $3;}'`.to_i * 1024
    end

    def get_free_space()
        `df -Pk #{@device} |grep ^/ | awk '{print $4;}'`.to_i * 1024
    end
    
    def get_free_space_percent()
        #total:100=used:x
        get_free_space()*100/get_total_space()
    end
end
{% endhighlight %}
