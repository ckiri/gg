REPORT_NAME = masterdata-gg-report

.PHONY: all pdf clean

all: pdf

build_dir: 
	@mkdir -p ./build

pdf: ./main.tex build_dir
	pdflatex \
		-output-directory=./build \
		-jobname=$(REPORT_NAME) main.tex 

clean: ./$(REPORT_NAME).pdf
	rm -r $(REPORT_NAME).pdf
