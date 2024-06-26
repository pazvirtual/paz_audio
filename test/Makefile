CXXVER := 17
MINMACOSVER := 10.12

ifeq ($(OS), Windows_NT)
    LIBPATH := /mingw64/lib
    OSPRETTY := Windows
else
    ifeq ($(shell uname -s), Darwin)
        OSPRETTY := macOS
    else
        OSPRETTY := Linux
    endif
    LIBPATH := /usr/local/lib
endif
CXXFLAGS := -std=c++$(CXXVER) -O3 -Wall -Wextra -Wno-missing-braces -Wold-style-cast
ifeq ($(OSPRETTY), macOS)
    CXXFLAGS += -mmacosx-version-min=$(MINMACOSVER) -Wunguarded-availability -Wno-string-plus-int
else
    ifeq ($(OSPRETTY), Windows)
        CXXFLAGS += -Wno-deprecated-copy
    endif
    CXXFLAGS += -pthread
endif
CXXFLAGS += -I..
LDLIBS := ../libpazaudio.a
ifeq ($(OSPRETTY), macOS)
    LDLIBS += -framework CoreAudio -framework AudioUnit
else
    ifeq ($(OSPRETTY), Linux)
        # Try a few locations for PortAudio.
        ifneq (, $(wildcard $(LIBPATH)/libportaudio.a))
            LDLIBS += $(LIBPATH)/libportaudio.a
        else ifneq (, $(wildcard /usr/lib/libportaudio.a))
            LDLIBS += /usr/lib/libportaudio.a
        else ifneq (, $(wildcard /usr/lib64/libportaudio.a))
            LDLIBS += /usr/lib64/libportaudio.a
        else ifneq (, $(wildcard /usr/lib/x86_64-linux-gnu/libportaudio.a))
            LDLIBS += /usr/lib/x86_64-linux-gnu/libportaudio.a
        else
            $(error Could not find "libportaudio.a".)
        endif
        LDLIBS += -lasound -ljack
    else
        LDLIBS += $(LIBPATH)/libportaudio.a $(LIBPATH)/libpthread.a -lwinmm -lole32 -lsetupapi
        LDFLAGS += -static-libgcc -static-libstdc++
    endif
endif

SRC := $(wildcard *.cpp)
EXE := $(SRC:.cpp=)

print-% : ; @echo $* = $($*)

.PHONY: $(SRC)

all: $(EXE)

ifeq ($(OSPRETTY), macOS)
%: %.cpp
	$(CXX) -arch arm64 -o $@_arm64 $< $(CXXFLAGS) $(INCL) $(LDFLAGS) $(LDLIBS)
	$(CXX) -arch x86_64 -o $@_x86_64 $< $(CXXFLAGS) $(INCL) $(LDFLAGS) $(LDLIBS)
	lipo -create -output $@ $@_arm64 $@_x86_64
endif

ifeq ($(OSPRETTY), macOS)
clean:
	$(RM) $(EXE) $(foreach n,$(EXE),$n_arm64 $n_x86_64)
else
clean:
	$(RM) $(EXE)
endif
