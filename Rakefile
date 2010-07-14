require 'uri'
require 'net/ftp'
require 'shellwords'

### Helpers

verbose false

# an Enumerable collection of URIs
@ftp_mirrors = %w[
  ftp://ftp.gnu.org/gnu/bash
  ftp://ftp.cwru.edu/pub/bash
].map { |url| URI.parse url }.each

module Bash
  def self.patchlevel
    File.read('patchlevel.h')[/^#define\s+PATCHLEVEL\s+(\d+)/, 1].to_i
  end
end

module Git
  def self.git
    @gitbin ||= %x(which git).chomp
    raise 'No git executable found!' unless File.executable? @gitbin
    @gitbin
  end

  # Branch names are major.minor version numbers
  def self.current_branch
    b = %x(#{self.git} symbolic-ref HEAD 2>/dev/null).chomp
    $?.exitstatus.zero? ? b[/^refs\/heads\/(.*)/, 1] : nil
  end
end

### Tasks

task :default => :patch

desc 'Apply patches from patch directory'
task :patch => 'download:patches' do
  patchlevel = Bash.patchlevel
  Dir['patches/*'].sort.each do |f|
    next if f[/-(\d+)$/, 1].to_i <= patchlevel
    warn "### Applying patch #{f}"
    system "patch -p0 --version-control never < #{f.shellescape}"
  end
end

namespace :download do
  desc 'Download patches for the current release'
  task :patches do
    mkdir_p 'patches'
    uri = @ftp_mirrors.next
    warn "### Downloading patches from #{uri.host}"
    Dir.chdir 'patches' do
      Net::FTP.open uri.host do |ftp|
        ftp.passive = true
        ftp.login
        ftp.nlst("#{uri.path}/bash-#{Git.current_branch}-patches").each do |f|
          next if File.extname(f) == '.sig' || File.exists?(File.basename f)
          warn "-> #{uri.host}/#{f}"
          ftp.get f
        end
      end
    end
  end
end
