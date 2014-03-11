require 'active_support/core_ext/string'

name = 'aps-spin-lifetime'
build_name = 'APS Spin Lifetime'
tex_src = 'tex'
builds = 'builds'

task :default => :build

desc 'Build LaTeX.'
task build: [:tex]

desc 'Compile LaTeX to PDF.'
task :tex do
  FileUtils.mkdir_p builds
  path = "#{tex_src}/#{name}.tex"

  fail RuntimeError, "Error: #{path} not found." unless File.exists? path

  Dir.chdir tex_src do
    system 'latexmk', '-xelatex', '-f', "#{name}.tex"
    FileUtils.mv "#{name}.pdf", "../#{builds}/#{build_name}.pdf"
  end
end

desc 'Clean out temporary LaTeX files.'
task :clean do
  Dir.chdir tex_src do
    system 'latexmk', '-c'
    %w(fls bbl bib).each do |ext|
      Dir.glob("*.#{ext}") { |f| File.unlink f }
    end
  end
end
