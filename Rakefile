# -*- ruby -*-

require 'tempfile'

task :default => :build

PANDOC_HTML_OPTS = [
  '-i',
  '--self-contained',
  '-t', 'slidy',
  '--smart'
]

PANDOC_PDF_OPTS = [
  '-t', 'beamer',
  '-V', 'theme:Berkeley',
  '--template', 'pdf-template.tex'
]

task :build => [:html, :pdf]

task :html => :css do
  tmp = Tempfile.new('slides')
  tmp.close
  begin
    args = PANDOC_HTML_OPTS + ['-s', 'slides.md', '-o', tmp.path]
    sh 'pandoc', *args
    puts "Inserting custom stylesheet and generating slides.html"
    File.open("slides.html", "w") do |out|
      File.open(tmp.path).each do |line|
        if line =~ %r{^</head>}
          custom_css = File.open('custom.css').read
          out.write <<-EOF
<style type="text/css">
#{custom_css}
</style>
          EOF
        end
        out.write(line)
      end
    end
  ensure
    tmp.unlink
  end
end

task :pdf do
  # Pandoc wants UTF-8, which is fine. But it uses LaTeX to generate PDF,
  # and LaTeX gags on NBSP characters. Filter them out.
  tmp = Tempfile.new('slides')
  tmp.close
  begin
    File.open(tmp.path, 'w') do |out|
      File.open('slides.md', encoding: 'utf-8').each do |line|
        out.write(line.gsub("\u00a0", " "))
      end
    end

    args = PANDOC_PDF_OPTS + [tmp.path, '-o', 'slides.pdf']
    sh 'pandoc', *args
  ensure
    tmp.unlink
  end
end

task :css => ['custom.css']

file 'custom.css' => ['custom.less'] do
  sh 'sh', '-c', 'lessc custom.less >custom.css'
end

task :clean do
  %w{combined.tex *.aux *.vrb *.out *.log custom.css
     slides.pdf slides.tex slides.html}.each do |pat|
    Dir.glob(pat).each do |i|
      rm_f i
    end
  end
end
