licensedir = $(datarootdir)/licenses/fontship
datadir = $(datarootdir)/fontship

bin_SCRIPTS = fontship
python_PYTHON = fontship.py
dist_doc_DATA = README.md CHANGELOG.md
dist_man_MANS = fontship.1
dist_license_DATA = LICENSE
dist_data_DATA = src/fontship.mk src/rules.mk src/functions.mk src/rules-glyphs.mk src/rules-sfd.mk src/rules-ufo.mk
EXTRA_DIST = .version Dockerfile

BUILT_SOURCES = .version

CLEANFILES = $(BUILT_SOURCES) $(bin_SCRIPTS)

.version: $(shell test -e .git && awk '{print ".git/" $$2}' .git/HEAD)
	mv $@{,-prev} 2>/dev/null || touch $@-prev
	( test -e .git && ./build-aux/git-version-gen .tarball-version || echo $(VERSION) ) > $@
	cmp -s $@{,-prev} || autoreconf configure.ac --force -W none

dist: fontship-$(VERSION).md

dist-hook:
	echo $(VERSION) > $(distdir)/.tarball-version

check: selfcheck

.PHONY: selfcheck
selfcheck: $(bin_SCRIPTS) | $(BUILT_SOURCES)
	./$< --version | grep -Ff $(firstword $|)

RELTYPE ?=

.PHONY: tagrelease
tagrelease:
	test -z $$(git tag --points-at HEAD) || exit 0 # end if we are already on a release tag
	git diff-index --quiet --cached HEAD || exit 1 # die if anything staged but not committed
	git diff-files --quiet || exit 1 # die if any tracked files have unstagged changes
	npm run release -- $(and $(RELTYPE),--release-as $(RELTYPE))

.PHONY: prerelease
prerelease: all

.PHONY: release-preview
release-preview:
	npm run release -- --dry-run $(and $(RELTYPE),--release-as $(RELTYPE))

.PHONY: release
release: tagrelease

_svg_cleanup = select-all;object-to-path;vacuum-defs
_scour_args = --quiet --set-precision=4 --remove-metadata --enable-id-stripping --strip-xml-prolog --strip-xml-space --no-line-breaks --no-renderer-workaround

.PHONY: media
media: media/logo.svg media/emblum.svg media/icon.svg media_badge

media/logo.svg: media/logo-source.svg
	inkscape --batch-process \
		--actions "select-by-id:F;EditDelete;$(_svg_cleanup)" \
		$< -o $@
	scour $(_scour_args) $@ /dev/stdout | sponge $@

media/emblum.svg: media/logo-source.svg
	inkscape --batch-process \
		--actions "select-by-id:Fontship;EditDelete;select-by-id:truck;FitCanvasToSelection;$(_svg_cleanup)" \
		$< -o $@
	scour $(_scour_args) $@ /dev/stdout | sponge $@

media/icon.svg: media/logo-source.svg
	inkscape --batch-process \
		--actions "select-by-id:F,Fontship;EditDelete;select-by-id:truck;FitCanvasToSelection;select-by-id:cargo;object-set-attribute:fill,#000;$(_svg_cleanup)" \
		$< -o $@
	scour $(_scour_args) $@ /dev/stdout | sponge $@

media/badge.svg: media/logo-source.svg
	inkscape --batch-process \
		--actions "select-by-id:F,Fontship;EditDelete;select-by-id:truck;FitCanvasToSelection;$(_svg_cleanup);select-all;object-set-attribute:stroke,#FFF;object-set-attribute:fill,#FFF" \
		$< -o $@
	scour $(_scour_args) $@ /dev/stdout | sponge $@

.PHONY: media_badge
media_badge: media/badge.svg
	@echo =================
	@svgo --datauri base64 --multipass --quiet -i $< -o -
	@echo =================

fontship-%.md: CHANGELOG.md
	sed -e '/\.\.\.v$*/,/\.\.\.v/!d' $< | \
		sed -e '1,3d;N;$$!P;$$!D;$$d' > $@

.PHONY: docker
docker: Dockerfile
	docker build \
		--build-arg VCS_REF="$(VERSION)" \
		--tag theleagueof/fontship:HEAD \
		./
