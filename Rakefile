require "rake/clean"
require "pathname"
require "rbconfig"
require "fileutils"

PROJECT_ROOT = File.expand_path(__dir__)

# ---- config

DIST_DIR = "#{PROJECT_ROOT}/dist"
SRC_DIR = "#{PROJECT_ROOT}/src"
FIG_DIR = "#{PROJECT_ROOT}/fig"
ETC_DIR = "#{PROJECT_ROOT}/etc"

ROOT_TEX = "#{PROJECT_ROOT}/root.tex"

TARGET = "#{DIST_DIR}/document.pdf"

module HostOS 
  class Base
    include Rake::DSL
    
    def preview_pdf(pdf)
      raise NotImplementedError.new
    end
    
    def fig_to_pdf(fig, pdf)
      case fig
      when /\.(eps|svg)$/i then   
        # vector image: convert with inkscape
        ->(){ sh "inkscape", "-f", fig, "-A", pdf }
      when /\.(jpe?g|png|bmp)/i then  
        # bitmap image: convert with ImageMagick
        ->(){ sh "convert", fig, pdf }
      end
    end
    
    def markdown_to_tex(md, tex)
      sh "pandoc", md, "-o", tex
    end
  end
  
  class Windows < Base
    def preview_pdf(pdf)
      sh "start", pdf
    end
  end
  
  class Mac < Base
    def preview_pdf(pdf)
      sh "open", pdf
    end
  end
  
  class Linux < Base
    def preview_pdf(pdf)
      sh "xdg-open", pdf
    end
  end
end

module TeXLive
  extend Rake::DSL
  extend self
  
  def tex_to_pdf(tex, pdf, latexmkrc)
    cd Pathname.new(tex).parent do
      sh "latexmk", "-r", latexmkrc, "-pdfdvi", File.basename(tex)
    end
  end
  
  def extractbb(pdf)
    cd Pathname.new(pdf).parent do
      sh "extractbb", File.basename(pdf)
    end
  end
end

# ---- host os

case RbConfig::CONFIG['host_os']
when /mswin|msys|mingw|cygwin|bccwin|wince|emc/
  HOST_OS = HostOS::Windows.new
when /darwin|mac os/
  HOST_OS = HostOS::Mac.new
when /linux/
  HOST_OS = HostOS::Linux.new
else
  raise "unknown os"
end
 
# ---- files

directory DIST_DIR
file TARGET => DIST_DIR

SRC = FileList[ROOT_TEX] + FileList["#{SRC_DIR}/**/*"]
TEX = SRC.select {|s| s.match(/\.(tex|sty)/i)}
MARKDOWN = SRC.select {|s| s.match(/\.(md|markdown)/i)}

FIG = FileList["#{FIG_DIR}/**/*.*"]
FIG_PDF = FIG.ext("pdf")
FIG_XBB = FIG_PDF.ext("xbb")
FIG.zip(FIG_PDF) {|src, dst|
  convert_cmd = HOST_OS.fig_to_pdf(src, dst)
  if convert_cmd and not src.match(/\.pdf$/)
    file dst => src do
      convert_cmd.call
    end
    CLEAN.include(dst)
  end
}

# ---- rules

rule ".pdf" => ".tex" do |task|
  TeXLive.tex_to_pdf(task.source, task.name, "#{ETC_DIR}/latexmkrc")
end

rule ".tex" => ".md" do |task|
  HOST_OS.markdown_to_tex(task.source, task.name)
end

rule ".xbb" => ".pdf" do |task|
  TeXLive.extractbb(task.source)
end

rule TARGET => ROOT_TEX.ext("pdf") do |task|
  Rake::Task[task.source].invoke("paty")
  FileUtils.mv(task.source, task.name)
end

# ---- tasks

task default: "preview"

desc "Build #{TARGET}"
task build: TARGET
file TARGET => FileList[TEX, FIG_PDF, FIG_XBB, MARKDOWN.ext("tex")]

desc "Preview #{TARGET}"
task preview: TARGET do |task|
  HOST_OS.preview_pdf(task.prerequisites[0])
end

desc "Convert all image files to pdf."
task fig_pdf: FIG_PDF

desc "Create xbb files from pdf files."
task fig_xbb: FIG_XBB

# ---- clean/clobber

CLEAN.include(FIG_XBB)
CLEAN.include(MARKDOWN.ext("tex"))
CLEAN.include(FileList[ROOT_TEX.ext("*")])
CLEAN.exclude(ROOT_TEX)
CLOBBER.include(TARGET)

CLEAN.existing!
CLEAN.uniq!
CLOBBER.existing!
CLOBBER.uniq!
