require 'active_support/core_ext/string'

name = 'aps-spin-lifetime'
build_name = 'APS Spin Lifetime'
tex_src = 'tex'
build = 'build'

task :default => :build

desc 'Build LaTeX.'
task build: [:tex]

desc 'Compile LaTeX to PDF.'
task :tex do
  build = File.expand_path build
  FileUtils.mkdir_p build
  path = File.join tex_src, "#{name}.tex"

  fail RuntimeError, "Error: #{path} not found." unless File.exists? path

  Dir.chdir tex_src do
    system 'latexmk', '-xelatex', '-f', "#{name}.tex"
    FileUtils.mv "#{name}.pdf", File.join(build, "#{build_name}.pdf")
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

desc 'Build a single LaTeX file for submission.'
task expand: [:clean] do
  out = File.join build, name
  out_file = File.join out, "#{name}.tex"
  FileUtils.remove_entry_secure out
  FileUtils.mkdir_p out
  Dir.chdir tex_src do
    system 'latexpand', '--keep-comments', '-o', '#{name}.expanded.tex', "#{name}.tex"
    FileUtils.mv '#{name}.expanded.tex', File.join('../', out_file)
    %w(spintronics software).each do |f|
      FileUtils.cp File.join('components', 'references', "#{f}.bib"), File.join('../', out)
    end
    %w(figures).each { |d| FileUtils.cp_r d, File.join('../', out) }
  end
  Dir.chdir out do
    Dir[File.join('figures', '*.tex')].each { |f| FileUtils.remove_entry_secure f }
    FileUtils.cp_r Dir['figures/*'], '.'
    FileUtils.remove_entry_secure 'figures'
  end
  text = File.read(out_file)
  text.gsub!('components/references/', '')
  text.gsub!('figures/', '')
  File.open(out_file, 'w') { |f| f.puts text }
end
