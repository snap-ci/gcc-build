require 'rubygems'
require 'bundler/setup'

require 'rake/clean'

distro = nil
fpm_opts = ""

if File.exist?('/etc/system-release') && File.read('/etc/redhat-release') =~ /centos|redhat|fedora|amazon/i
  distro = 'rpm'
  fpm_opts << " --rpm-user root --rpm-group root "
elsif File.exist?('/etc/os-release') && File.read('/etc/os-release') =~ /ubuntu|debian/i
  distro = 'deb'
  fpm_opts << " --deb-user root --deb-group root "
end

unless distro
  $stderr.puts "Don't know what distro I'm running on -- not sure if I can build!"
end

base_url = "http://mirrors-usa.go-parts.com/gcc/releases"

%w(4.6.4 4.7.4 4.8.3).each do |version|
  namespace version do
    release = Time.now.utc.strftime('%Y%m%d%H%M%S')

    description_string = %Q{Collection of compilers for gcc and g++}

    jailed_root = File.expand_path('../jailed-root', __FILE__)
    prefix = File.join('/opt/local/gcc', version)

    CLEAN.include("downloads")
    CLEAN.include("jailed-root")
    CLEAN.include("log")
    CLEAN.include("pkg")
    CLEAN.include("src")

    task :init do
      mkdir_p "log"
      mkdir_p "pkg"
      mkdir_p "src"
      mkdir_p "downloads"
      mkdir_p "jailed-root"
    end

    task :download do
      cd 'downloads' do
        sh("curl --fail #{base_url}/gcc-#{version}/gcc-#{version}.tar.gz > gcc-#{version}.tar.gz 2>/dev/null")
        sh("curl --fail #{base_url}/gcc-#{version}/md5.sum | grep gcc-#{version}.tar.gz > gcc-#{version}.tar.gz.md5sum 2>/dev/null")
        sh("md5sum --status --check gcc-#{version}.tar.gz.md5sum")
      end
    end

    task :configure do
      sh("sudo yum install --assumeyes mpfr-devel libmpc-devel glibc-devel")
      cd "src" do
        sh "tar -zxf ../downloads/gcc-#{version}.tar.gz"
        cd "gcc-#{version}" do
          sh "./configure --prefix=#{prefix} --disable-multilib --enable-languages=c,c++,lto > #{File.dirname(__FILE__)}/log/configure.#{version}.log 2>&1"
        end
      end
    end

    task :make do
      num_processors = %x[grep --count processor /proc/cpuinfo].chomp.to_i
      num_jobs       = num_processors + 1

      cd "src/gcc-#{version}" do
        sh("make -j#{num_jobs} > #{File.dirname(__FILE__)}/log/make.#{version}.log 2>&1")
      end
    end

    task :make_install do
      rm_rf  jailed_root
      mkdir_p jailed_root
      cd "src/gcc-#{version}" do
        sh("make install DESTDIR=#{jailed_root} > #{File.dirname(__FILE__)}/log/make-install.#{version}.log 2>&1")
      end
    end

    task :fpm do
      cd 'pkg' do
        sh(%Q{
             bundle exec fpm -s dir -t #{distro} --name gcc-#{version} -a x86_64 --version "#{version}" -C #{jailed_root} --directories #{prefix} --verbose #{fpm_opts} --maintainer snap-ci@thoughtworks.com --vendor snap-ci@thoughtworks.com --url http://snap-ci.com --description "#{description_string}" --iteration #{release} --license 'GPL' .
        })
      end
    end

    desc "build and package gcc-#{version}"
    task :all => [:clean, :init, :download, :configure, :make, :make_install, :fpm]
  end

  task :default => "#{version}:all"
end

desc "build all gcc"
task :default