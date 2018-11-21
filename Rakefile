ROOT    = File.dirname(__FILE__)
VERSION = "2.4.5"

require "erb"
require 'fileutils'
require "tmpdir"

def tempdir
  Dir.mktmpdir do |dir|
    Dir.chdir(dir) do
      yield(dir)
    end
  end
end

task :default => "pkg/ruby-#{VERSION}.pkg"

file "pkg/ruby-#{VERSION}.pkg" => "/usr/local/td/ruby" do |t|
  tempdir do |dir|
    cp_r t.prerequisites.first, "#{dir}/ruby"

    kbytes = %x{ du -ks ruby | cut -f 1 }
    num_files = %x{ find ruby | wc -l }

    mkdir_p "pkg"
    mkdir_p "pkg/Resources"
    mkdir_p "pkg/ruby-#{VERSION}.pkg"

    dist = File.read("#{ROOT}/support/Distribution.erb")
    dist = ERB.new(dist).result(binding)
    File.open("pkg/Distribution", "w") { |f| f.puts dist }

    dist = File.read("#{ROOT}/support/PackageInfo.erb")
    dist = ERB.new(dist).result(binding)
    File.open("pkg/ruby-#{VERSION}.pkg/PackageInfo", "w") { |f| f.puts dist }

    sh %{ mkbom -s ruby pkg/ruby-#{VERSION}.pkg/Bom }

    Dir.chdir("ruby") do
      sh %{ pax -wz -x cpio . > ../pkg/ruby-#{VERSION}.pkg/Payload }
    end

    mkdir_p "#{ROOT}/pkg"
    sh %{ pkgutil --flatten pkg #{ROOT}/#{t.name} }
  end
end

$TD_RUBY_DIR           = '/usr/local/td/ruby'
$READLINE_PACKAGE_NAME = 'readline-6.3'
$OPENSSL_PACKAGE_NAME  = 'openssl-1.0.2q'

file "/usr/local/td/ruby" => 'setup_readline_and_openssl' do |t|
  sh %!CC='/usr/bin/clang' RUBY_CONFIGURE_OPTS="--with-out-ext=tk,dbm,gdbm,sdbm --with-readline-dir=#{$TD_RUBY_DIR} --with-openssl-dir=#{$TD_RUBY_DIR}" vendor/ruby-build/bin/ruby-build #{VERSION} #{t.name}!
end

task 'setup_readline_and_openssl' do
  Dir.chdir('./ext') do
    unless Dir.exist?("./#{$READLINE_PACKAGE_NAME}")
      sh "tar zxvf #{$READLINE_PACKAGE_NAME}.tar.gz"
      Dir.chdir("./#{$READLINE_PACKAGE_NAME}") do
        sh "CC='/usr/bin/clang' ./configure --prefix=#{$TD_RUBY_DIR} CC=/usr/bin/clang"
        sh "make"
        sh "make install"
      end
    end

    unless Dir.exist?("./#{$OPENSSL_PACKAGE_NAME}")
      unless File.exist?("#{$OPENSSL_PACKAGE_NAME}.tar.gz")
        system("curl -sJLO https://www.openssl.org/source/#{$OPENSSL_PACKAGE_NAME}.tar.gz")
      end
      sh "tar zxvf #{$OPENSSL_PACKAGE_NAME}.tar.gz"
      Dir.chdir("./#{$OPENSSL_PACKAGE_NAME}") do
        config = "--prefix=#{$TD_RUBY_DIR} zlib-dynamic no-ssl2 shared enable-cms"
        arch = "darwin64-x86_64-cc enable-ec_nistp_64_gcc_128"
        sh "CC='/usr/bin/clang' ./Configure #{config} #{arch}"
        sh "make -j 1"
        sh "make install_sw"
      end
    end
  end

  Dir.chdir($TD_RUBY_DIR) do
    FileUtils.rm_rf ["./bin/c_rehash", "./bin/openssl", "./ssl", "./share/readline"]
  end
end

task :clean do
  sh "rm -rf ./pkg ./ext/#{$OPENSSL_PACKAGE_NAME} ./ext/#{$READLINE_PACKAGE_NAME} /usr/local/td/ruby"
end
