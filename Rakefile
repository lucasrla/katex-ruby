# frozen_string_literal: true
require 'bundler/gem_tasks'
require 'rspec/core/rake_task'

RSpec::Core::RakeTask.new(:spec)

task :update, :version do |_task, args|
  require 'fileutils'
  require 'open-uri'

  # Download KaTeX
  version = args[:version]
  unless version
    $stderr.puts 'Specify KaTeX version, e.g. rake update[v0.7.0]'
    exit 64
  end
  dl_path = File.join('tmp', 'katex-dl', version)
  FileUtils.mkdir_p(dl_path)
  archive_path = File.join(dl_path, 'katex.tar.gz')
  unless File.exist?(archive_path)
    url = 'https://github.com/Khan/KaTeX/releases/download/' \
          "#{version}/katex.tar.gz"
    IO.copy_stream(open(url), archive_path)
  end
  katex_path = File.join(File.dirname(archive_path), 'katex')
  unless File.directory?(katex_path)
    system 'tar', 'xf', archive_path, '-C', File.dirname(archive_path)
  end

  # Copy assets
  assets_path = File.join('vendor', 'assets')
  FileUtils.rmdir assets_path
  FileUtils.mkdir_p assets_path
  FileUtils.cp_r File.join(katex_path, 'fonts'), assets_path
  FileUtils.mkdir_p File.join(assets_path, 'javascripts')
  FileUtils.cp File.join(katex_path, 'katex.min.js'),
               File.join(assets_path, 'javascripts', 'katex.js')
  FileUtils.mkdir_p File.join(assets_path, 'stylesheets')
  FileUtils.cp File.join(katex_path, 'katex.min.css'),
               File.join(assets_path, 'stylesheets', 'katex.css')

  # Create an sprockets version of katex CSS that is a Sass file that
  # uses asset-path for referencing fonts.
  sprockets_css_path = File.join(assets_path, 'sprockets', 'stylesheets')
  FileUtils.mkdir_p sprockets_css_path
  File.write(File.join(sprockets_css_path, 'katex.scss'),
             File.read(File.join(katex_path, 'katex.css'))
                 .gsub(%r{url\((['"]?)fonts/}, 'asset-path(\1'))

  # Update KATEX_VERSION in version.rb
  File.write('lib/katex/version.rb',
             File.read('lib/katex/version.rb')
                 .gsub(/KATEX_VERSION = '.*?'/, "KATEX_VERSION = '#{version}'"))
end

task default: :spec
