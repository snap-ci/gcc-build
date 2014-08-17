This repo contains scripts needed to build deb and rpms for various versions of gcc

# CentOS/RHEL

<pre>
$ yum install -y ruby rubygems ruby-devel rpm-build rpmdevtools readline-devel ncurses-devel gdbm-devel tcl-devel openssl-devel db4-devel byacc gcc libffi-devel libffi libxml2-devel libxslt-devel
$ gem install bundler rake --no-ri --no-rdoc
$ bundle install --path .bundle
</pre>

# Ubuntu

<pre>
$ apt-get install build-essentials dpkg-dev
$ bundle install --path .bundle
</pre>

# How to build one of the packages

<pre>
$ rake
</pre>
