include ../common.mak

LOADLIBES = -lm
override CFLAGS+= -I../src -I../b_overdrive -I../b_reverb -I../b_whirl
override CFLAGS+=`pkg-config --cflags lv2` -DLV2SYNTH

ifeq ($(IS_OSX), yes)
  FONTFILE?=/usr/X11/lib/X11/fonts/TTF/VeraBd.ttf
else
  FONTFILE?=/usr/share/fonts/truetype/ttf-bitstream-vera/VeraBd.ttf
endif

ifeq ($(shell test -f $(FONTFILE) || echo no ), no)
  $(warning UI font can not be found on this system, install bitstream-vera TTF or set the FONTFILE variable to a ttf file)
  $(warning LV2 GUI will not be built)
  FONT_FOUND=no
else
  FONT_FOUND=yes
endif

ifeq ($(IS_OSX), yes)
  HAVE_UI=$(shell pkg-config --exists ftgl && echo $(FONT_FOUND))
else
  HAVE_UI=$(shell pkg-config --exists glu ftgl && echo $(FONT_FOUND))
endif

targets=

LV2NAME=b_synth
LV2GUI=b_synthUI
TX=textures/

ifeq ($(LV2AVAIL), yes)
  targets+=lv2

  ifeq ($(HAVE_UI), yes)
    UICFLAGS=-I..
    UIDEPS=../pugl/pugl.h ../pugl/pugl_internal.h ui_model.h
    UIDEPS+=$(TX)drawbar.c $(TX)wood.c $(TX)dial.c
    UIDEPS+=$(TX)btn_vibl.c $(TX)btn_vibu.c $(TX)btn_overdrive.c $(TX)btn_perc_volume.c
    UIDEPS+=$(TX)btn_perc.c $(TX)btn_perc_decay.c $(TX)btn_perc_harmonic.c
    UIDEPS+=$(TX)bg_right_ctrl.c $(TX)bg_left_ctrl.c $(TX)bg_leslie_drum.c $(TX)bg_leslie_horn.c
    UIDEPS+=$(TX)help_screen_image.c
    UIDEPS+=$(TX)ui_button_image.c $(TX)ui_proc_image.c
    UIDEPS+=$(TX)uim_background.c $(TX)uim_cable1.c $(TX)uim_cable2.c $(TX)uim_caps.c
    UIDEPS+=$(TX)uim_tube1.c $(TX)uim_tube2.c
    ifeq ($(IS_OSX), yes)
      UIDEPS+=../pugl/pugl_osx.m
      UILIBS=../pugl/pugl_osx.m -framework Cocoa -framework OpenGL
      UI_TYPE=CocoaUI
    else
      UIDEPS+=../pugl/pugl_x11.c
      override CFLAGS+=`pkg-config --cflags glu`
      UILIBS=../pugl/pugl_x11.c -lX11 `pkg-config --libs glu`
      UI_TYPE=X11UI
    endif
    override CFLAGS+=`pkg-config --cflags ftgl` -DHAVE_FTGL
    UILIBS+=`pkg-config --libs ftgl`
    override CFLAGS+=-DFONTFILE=\"$(FONTFILE)\"
    targets+=$(LV2GUI)$(LIB_EXT)
  else
    $(warning "openGL/GLU is not available - install glu-dev to include LV2 GUI")
  endif
endif

#NOTE: midi.c and cfgParser.c needs to be re-compiled w/o HAVE_ASEQ
# and HAVE_ZITACONVOLVE. Other objects are identical.
OBJ= \
  ../src/midi.c \
  ../src/cfgParser.c \
  ../src/program.c \
  ../src/vibrato.c \
  ../src/state.c \
  ../src/tonegen.c \
  ../src/pgmParser.c \
  ../src/memstream.c \
  ../b_whirl/eqcomp.c \
  ../b_whirl/whirl.c \
  ../b_overdrive/overdrive.c \
  ../b_reverb/reverb.c \

all: $(targets)

lv2: manifest.ttl $(LV2NAME)$(LIB_EXT) $(LV2NAME).ttl

manifest.ttl: manifest.ttl.in manifest.ui.ttl.in
	sed "s/@LV2NAME@/$(LV2NAME)/;s/@LIB_EXT@/$(LIB_EXT)/" manifest.ttl.in > manifest.ttl
ifeq ($(HAVE_UI), yes)
	sed "s/@LV2NAME@/$(LV2NAME)/;s/@LV2GUI@/$(LV2GUI)/;s/@LIB_EXT@/$(LIB_EXT)/;s/@UI_TYPE@/$(UI_TYPE)/" manifest.ui.ttl.in >> manifest.ttl
endif

$(LV2NAME).ttl: $(LV2NAME).ttl.in $(LV2NAME).ui.ttl.in
	cat $(LV2NAME).ttl.in > $(LV2NAME).ttl
ifeq ($(HAVE_UI), yes)
	sed "s/@UI_TYPE@/$(UI_TYPE)/;s/@UI_REQ@/$(LV2UIREQ)/;" $(LV2NAME).ui.ttl.in >> $(LV2NAME).ttl
endif

$(LV2NAME)$(LIB_EXT): lv2.c $(OBJ) uris.h ../src/cfgParser.h ../src/midi.h
	$(CC) $(CPPFLAGS) $(CFLAGS) \
		-o $(LV2NAME)$(LIB_EXT) \
		lv2.c $(OBJ) \
	  -shared $(LV2LDFLAGS) $(LDFLAGS) $(LOADLIBES)

$(LV2GUI)$(LIB_EXT): ui.c uris.h $(UIDEPS)
	$(CC) $(CPPFLAGS) $(CFLAGS) $(UICFLAGS) \
		-o $(LV2GUI)$(LIB_EXT) ui.c \
		-shared $(LV2LDFLAGS) $(LDFLAGS) $(UILIBS)

install: $(targets)
ifeq ($(LV2AVAIL), yes)
	install -d $(DESTDIR)$(lv2dir)/$(LV2NAME)
	install -m755 $(LV2NAME)$(LIB_EXT) $(DESTDIR)$(lv2dir)/$(LV2NAME)
	install -m644 manifest.ttl $(LV2NAME).ttl $(DESTDIR)$(lv2dir)/$(LV2NAME)
  ifeq ($(HAVE_UI), yes)
	install -m755 $(LV2GUI)$(LIB_EXT) $(DESTDIR)$(lv2dir)/$(LV2NAME)
  endif
endif

uninstall:
	rm -f $(DESTDIR)$(lv2dir)/$(LV2NAME)/*.ttl
	rm -f $(DESTDIR)$(lv2dir)/$(LV2NAME)/$(LV2NAME)$(LIB_EXT)
	rm -f $(DESTDIR)$(lv2dir)/$(LV2NAME)/$(LV2GUI)$(LIB_EXT)
	-rmdir $(DESTDIR)$(lv2dir)/$(LV2NAME)

clean:
	rm -f *.o manifest.ttl b_synth.ttl
	rm -f $(LV2NAME)$(LIB_EXT)
	rm -f $(LV2GUI)$(LIB_EXT)

.PHONY: clean all install uninstall lv2
