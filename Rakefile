require 'uri'
require 'net/ftp'
require 'shellwords'

### Helpers

verbose false

def ftp_mirrors
  @ftp_mirrors ||= %w[
    ftp://ftp.gnu.org/gnu/bash
    ftp://ftp.cwru.edu/pub/bash
  ].map { |url| URI.parse url }
end

def patchlevel
  @patchlevel ||= File.read('patchlevel.h')[/^#define\s+PATCHLEVEL\s+(\d+)/, 1].to_i
end

def current_branch
  @current_branch ||= begin
    b = %x(git symbolic-ref HEAD 2>/dev/null).chomp
    $?.exitstatus.zero? ? b[/^refs\/heads\/(.*)/, 1] : nil
  end
end

### Tasks

task :default => :configure

desc 'Configure bash'
task :configure do
  cmd = %W[
    #{File.expand_path 'configure'}
    --prefix=#{ENV['PREFIX'] || '/opt/bash'}
  ]

  puts cmd.join(' ')
  system *cmd
end

desc 'Apply patches from patch directory'
task :patch => 'patch:download' do
  Dir['patch/*'].sort.each do |f|
    next if f[/-(\d+)$/, 1].to_i <= patchlevel
    warn "### Applying #{f}"
    system "patch -p0 < #{f.shellescape}"
  end
end

namespace :patch do
  desc 'Download patches for the current release'
  task :download do
    list = ftp_mirrors.dup
    uri  = list.shift

    begin
      warn "### Downloading patches from #{uri.host}"
      mkdir_p 'patch'

      Dir.chdir 'patch' do
        Net::FTP.open uri.host do |ftp|
          ftp.passive = true
          ftp.login
          ftp.nlst("#{uri.path}/bash-#{current_branch}-patches").each do |f|
            next if File.extname(f) == '.sig' or File.exists? File.basename f
            warn "-> #{uri.host}/#{f}"
            ftp.get f
          end
        end
      end
    rescue
      uri = list.shift
      retry if uri
    end
  end
end
