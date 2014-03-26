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
  FileUtils.mkdir_p build
  path = File.join tex_src, "#{name}.tex"

  fail RuntimeError, "Error: #{path} not found." unless File.exists? path

  Dir.chdir tex_src do
    system 'latexmk', '-xelatex', '-f', "#{name}.tex"
    FileUtils.mv "#{name}.pdf", File.join('../', build, "#{build_name}.pdf")
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
  out = "#{build}/#{name}"
  out_file = "#{out}/#{name}.tex"
  FileUtils.mkdir_p out
  Dir.chdir tex_src do
    system 'latexpand', '--keep-comments', '-o', "../#{out_file}", "#{name}.tex"
    %w(spintronics software).each do |f|
      FileUtils.cp "components/references/#{f}.bib", "../#{out}"
    end
    %w(figures).each { |d| FileUtils.cp_r d, "../#{out}"}
  end
  Dir["#{out}/figures/*.tex"].each { |f| FileUtils.remove_entry_secure f }
  text = File.read(out_file).gsub('components/references/', '')
  File.open(out_file, 'w') {|f| f.puts text}
end
