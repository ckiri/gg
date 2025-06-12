.PHONY: all slides

all: slides

pdf: ./slides.md
	npx @marp-team/marp-cli@latest ./slides.md -o ./slides.pdf

slides: ./slides.md
	npx @marp-team/marp-cli@latest ./slides.md -o ./slides.html

clean: ./scc25-slides.pdf
	rm -r ./slides.pdf
