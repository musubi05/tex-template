require 'rake/clean'
require 'pathname'

task default: 'open'

# dist directory
DIST_DIR = 'dist'
directory DIST_DIR

# target
TARGET_PDF = "#{DIST_DIR}/thesis.pdf"
file TARGET_PDF => DIST_DIR
CLOBBER.include(TARGET_PDF)

# sources
SRC_DIR = 'src'
SOURCES = FileList["#{SRC_DIR}/**/*"]
TEX = SOURCES.select {|s| s.match(/\.(tex|sty)/i)}
MARKDOWN = SOURCES.select {|s| s.match(/\.m(ark)?d(own)?/i)}

# create task to convert all images file to pdf files
FIG_DIR = "#{SRC_DIR}/fig"
FIG = FileList["#{FIG_DIR}/**/*"].map {|source|
  pdf = source.ext('pdf')

  command = 
    case source
    when /\.pdf$/i then ->{}
    when /\.eps$/i then ->{sh 'epstopdf', source}
    when /\.(jpe?g|png|svg|bmp)/i then ->{sh 'convert', source, pdf}
    end

  unless command
    next
  end
  
  unless source.match(/\.pdf$/)
    file pdf => source do
      command.call
    end
    CLEAN.include(pdf)
  end
  
  pdf
}.select{|f| f}
XBB = FIG.ext('xbb')

desc 'Build target pdf file.'
task build: TARGET_PDF
file TARGET_PDF => FileList[TEX, FIG, XBB, MARKDOWN.ext('tex')]

desc 'Open target pdf file.'
task open: TARGET_PDF do |task|
  sh 'evince', task.prerequisites[0]
end

desc 'Convert any image file to pdf file.'
task fig: FIG

desc 'Create xbb files from pdf files.'
task xbb: XBB

rule '.xbb' => '.pdf' do |task|
  sh 'extractbb', task.source
end
CLEAN.include(XBB)

rule '.tex' => '.md' do |task|
  sh 'pandoc', task.source, '-o', task.name
end
CLEAN.include(MARKDOWN.ext('tex'))

rule '.pdf' => '.tex' do |task|
  basename = File.basename(task.source)
  cd Pathname.new(task.name).parent do
    sh 'platex', '-kanji=UTF8', basename
    sh 'platex', '-kanji=UTF8', basename
    sh 'dvipdfmx', basename.ext('dvi')
  end
end
CLEAN.include(TEX.ext('dvi'), TEX.ext('aux'), TEX.ext('log'), TEX.ext('xbb'))

rule %r{^#{DIST_DIR}/.+\.pdf} => "%{^#{DIST_DIR},#{SRC_DIR}}X.pdf" do |task|
  Rake::Task[task.source].invoke('paty')
  sh 'mv', task.source, task.name
end

CLEAN.existing!
CLEAN.uniq!
CLOBBER.existing!
CLOBBER.uniq!
