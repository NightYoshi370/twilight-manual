ifeq ($(OS),Windows_NT)
CHROMIUM	:= chromium.exe
SED			:= sed.exe -i
else
	ifeq ($(shell uname -s),Darwin)
# I use Edge, change this if using another Chromium browser -Pk11
CHROMIUM	:= /Applications/Microsoft\ Edge.app/Contents/MacOS/Microsoft\ Edge
# macOS sed is dumb apparently
SED			:= sed -i ''
	else
CHROMIUM	:= chromium
SED			:= sed -i
	endif
endif

# Acholi is the default for Crowdin's In-Context translations
EXCLUDE		:= Acholi assets
LANGS		:= $(filter-out $(EXCLUDE),$(subst /,,$(wildcard */)))
NITROFILES	:= ../nitrofiles

INPUTS		:= $(foreach dir,$(LANGS),$(wildcard $(dir)/*.html))
TARGETS		:= $(foreach file,$(INPUTS:.html=.gif),$(addprefix $(NITROFILES)/pages/, $(file)))

INIFILES	:= $(wildcard $(NITROFILES)/pages/english/*.ini)
INITARGETS	:= $(foreach file,$(INIFILES),$(foreach lang,$(filter-out english,$(LANGS)),$(subst english,$(lang),$(file))))

.PHONY: all clean server killserver

all:	inifiles server $(TARGETS) killserver

server:
	python3 -m http.server 1>/dev/null 2>&1 & echo $$! > server.pid

killserver:
	if [ -f server.pid ]; then kill $$(cat server.pid); rm server.pid; fi

$(NITROFILES)/pages/%.gif	:	%.html
	@echo $$(basename $(@D))/$$(basename $<)
	@[ -d "$(@D)" ] || mkdir -p "$(@D)"
	$(CHROMIUM) --no-sandbox --hide-scrollbars --headless --disable-gpu --window-size=256,3000 1>/dev/null 2>&1 --screenshot$(SCREENSHOT) http://127.0.0.1:8000/$<
	@ffmpeg -i screenshot.png -vf "palettegen=max_colors=246" palette.png -y -loglevel error
	@ffmpeg -i screenshot.png -i palette.png -filter_complex "$$(ffmpeg -loop 1 -i screenshot.png -frames:v 3 -vf 'negate,cropdetect=0:2:0' -f null - 2>&1 | awk '/y2/ {print $$NF}' | tail -1),paletteuse" $@ -y -loglevel error
	@rm -f screenshot.png palette.png

inifiles	:	$(INITARGETS)

$(foreach lang,$(filter-out english,$(LANGS)),$(NITROFILES)/pages/$(lang)/%.ini)	:	$(NITROFILES)/pages/english/%.ini
	@echo $*.ini
	@$(foreach lang,$(filter-out english,$(LANGS)),[ -d "$(NITROFILES)/pages/$(lang)" ] || mkdir -p "$(NITROFILES)/pages/$(lang)";)
	@$(foreach lang,$(filter-out english,$(LANGS)),cp $< $(NITROFILES)/pages/$(lang)/$*.ini;)
	@$(foreach lang,$(LANGS),$(SED) "s/TITLE = .*/TITLE = $(shell xmllint --html $(lang)/$*.html --xpath "//head/title/text()")/" $(NITROFILES)/pages/$(lang)/$*.ini;)

clean:
	@echo clean ...
	@rm -rf $(TARGETS)
	@rm -rf $(INITARGETS)
