require 'active_support/core_ext/string'
require 'yaml'

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
    %w(fls bbl bib pdf xml).each do |ext|
      Dir["*.#{ext}"].each { |f| File.unlink f }
      Dir["figures/*.#{ext}"].each { |f| File.unlink f }
    end
  end
end

desc "Build a single LaTeX file for submission (for use with pdfLaTeX).\
  \nWill be placed in build/aps-spin-lifetime.\
  \nAn archive will be created at build/aps-spin-lifetime.tar.gz."
task expand: [:clean] do
  tex_map = 'map.pdflatex.yml'
  bib_map = File.join tex_src, 'components', 'references', 'map.pdflatex.yml'
  maps = {}
  maps[:tex] = YAML.load_file tex_map if File.exists? tex_map
  maps[:bib] = YAML.load_file bib_map if File.exists? bib_map

  tex_src_path = File.expand_path tex_src
  out = File.expand_path File.join(build, name)
  out_file = File.expand_path File.join(out, "#{name}.tex")
  tar_file = File.expand_path File.join(build, "#{name}.tar.gz")
  FileUtils.remove_entry_secure tar_file if File.exists? tar_file
  FileUtils.remove_entry_secure out if Dir.exists? out
  FileUtils.mkdir_p out

  Dir.mktmpdir do |dir|
    FileUtils.cp_r tex_src_path, dir

    Dir.chdir File.join(dir, tex_src) do
      src_file = File.expand_path "#{name}.tex"

      text = File.read src_file
      text.gsub! '_preamble.xelatex', '_preamble.pdflatex'
      text.gsub! '_header.xelatex', '_header.pdflatex'
      File.open(src_file, 'w') { |f| f.puts text }

      system 'latexpand', '--keep-comments', '-o', "#{name}.expanded.tex", src_file
      FileUtils.mv "#{name}.expanded.tex", out_file
      %w(spintronics software).each do |f|
        FileUtils.cp File.join('components', 'references', "#{f}.bib"), out
      end
      %w(figures).each { |d| FileUtils.cp_r d, out }
    end
  end
  Dir.chdir out do
    Dir[File.join('figures', '*.tex')].each { |f| FileUtils.remove_entry_secure f }
    FileUtils.cp_r Dir['figures/*'], '.'
    FileUtils.remove_entry_secure 'figures'

    text = File.read out_file
    text.gsub! 'components/references/', ''
    text.gsub! 'figures/', ''
    maps[:tex].each { |m| text.gsub! m[0], m[1] } if maps[:tex]
    File.open(out_file, 'w') { |f| f.puts text }

    Dir["*.bib"].each do |bib_file|
      text = File.read bib_file
      maps[:bib].each { |m| text.gsub! m[0], m[1] }
      File.open(bib_file, 'w') { |f| f.puts text }
    end if maps[:bib]

    system 'latexmk', '-pdf', "#{name}.tex"
    system 'latexmk', '-c'
    system 'latexpand', '--keep-comments', '--expand-bbl', "#{name}.bbl", '-o', "#{name}.expanded.tex", out_file
    FileUtils.mv "#{name}.expanded.tex", out_file

    %w(pdf bib bbl).each do |ext|
      Dir["*.#{ext}"].each { |f| File.unlink f }
    end
  end

  system 'tar', '-cz', '-f', tar_file, '-C', build, name
end
