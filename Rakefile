require 'rake/clean'
require 'pathname'
require 'rbconfig'

# --- config

DIST_DIR = 'dist'
SRC_DIR = 'src'
FIG_DIR = "fig"

ROOT_TEX = 'root.tex'
TARGET_BASENAME = "document.pdf"

case RbConfig::CONFIG['host_os']
when /darwin|mac os/
  PREVIEW = ->(pdf){sh 'open', pdf}
when /linux/
  PREVIEW = ->(pdf){sh 'xdg-open', pdf}
else
  raise 'assert'
end
  
FIG_TO_PDF = ->(src, dst){
  case src
  when /\.eps$/i then 
    ->{sh 'epstopdf', src, "--outfile='#{dst}'"}
  when /\.(jpe?g|png|svg|bmp)/i then 
    ->{sh 'convert', src, dst}
  end
}

rule '.pdf' => '.tex' do |task|
  basename = File.basename(task.source)
  cd Pathname.new(task.name).parent do
    sh 'latexmk', '-r', 'etc/latexmkrc', '-pdfdvi', basename
  end
end

rule '.tex' => '.md' do |task|
  sh 'pandoc', task.source, '-o', task.name
end

rule '.xbb' => '.pdf' do |task|
  sh 'extractbb', task.source
end


# ---- files

directory DIST_DIR
TARGET = "#{DIST_DIR}/#{TARGET_BASENAME}"
file TARGET => DIST_DIR

SRC = FileList[ROOT_TEX] + FileList["#{SRC_DIR}/**/*"]
TEX = SRC.select {|s| s.match(/\.(tex|sty)/i)}
MARKDOWN = SRC.select {|s| s.match(/\.m(ark)?d(own)?/i)}

FIG = FileList["#{FIG_DIR}/**/*"]
FIG_PDF = FIG.ext('pdf')
FIG_XBB = FIG_PDF.ext('xbb')
FIG.zip(FIG_PDF) {|src, dst|
  convert_cmd = FIG_TO_PDF[src, dst]
  if convert_cmd and not src.match(/\.pdf$/)
    file dst => src do
      convert_cmd.call
    end
    CLEAN.include(dst)
  end
}


# ---- tasks

task default: 'preview'

desc "Build #{TARGET}"
task build: TARGET
file TARGET => FileList[TEX, FIG_PDF, FIG_XBB, MARKDOWN.ext('tex')]

desc "Preview #{TARGET}"
task preview: TARGET do |task|
  PREVIEW[task.prerequisites[0]]
end

desc 'Convert all image files to pdf.'
task fig_pdf: FIG_PDF

desc 'Create xbb files from pdf files.'
task fig_xbb: FIG_XBB

rule TARGET => ROOT_TEX.ext('pdf') do |task|
  Rake::Task[task.source].invoke('paty')
  sh 'mv', task.source, task.name
end


# ---- clean

CLEAN.include(FIG_XBB)
CLEAN.include(MARKDOWN.ext('tex'))
CLEAN.include(FileList[ROOT_TEX.ext('*')])
CLEAN.exclude(ROOT_TEX)
CLOBBER.include(TARGET)

CLEAN.existing!
CLEAN.uniq!
CLOBBER.existing!
CLOBBER.uniq!
