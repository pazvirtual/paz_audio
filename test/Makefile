CXXVER := 17
MINMACOSVER := 11.0

ifeq ($(OS), Windows_NT)
    OSPRETTY := Windows
    ifeq ($(MSYSTEM), UCRT64)
        CC := gcc
        CXX := g++
        LIBPATH := /ucrt64/lib
    else ifeq ($(MSYSTEM), CLANG64)
        CC := clang
        CXX := clang++
        LIBPATH := /clang64/lib
    else
        $(error Unsupported Windows environment)
    endif
else
    ifeq ($(shell uname -s), Darwin)
        OSPRETTY := macOS
        CC := clang
        CXX := clang++
    else
        OSPRETTY := Linux
        CC := gcc
        CXX := g++
    endif
    LIBPATH := /usr/local/lib
endif
CXXFLAGS := -std=c++$(CXXVER) -O3 -Wall -Wextra -Wno-missing-braces \
    -Wold-style-cast
ifeq ($(OSPRETTY), macOS)
    CXXFLAGS += -mmacosx-version-min=$(MINMACOSVER) -Wunguarded-availability
else ifeq ($(OSPRETTY), Linux)
    CXXFLAGS += -pthread
endif
ifeq ($(CC), clang)
    CXXFLAGS += -Wno-string-plus-int
endif
CXXFLAGS += -I..
LDLIBS := ../libpazaudio.a
ifeq ($(OSPRETTY), macOS)
    LDLIBS += -framework CoreAudio -framework AudioUnit
else ifeq ($(OSPRETTY), Windows)
    LDLIBS += $(LIBPATH)/libportaudio.a -lwinmm -lole32 -lsetupapi
    ifeq ($(CC), gcc)
        LDLIBS += -Wl,-Bstatic -lstdc++ -lpthread -Wl,-Bdynamic
    else
        LDLIBS += -Wl,-Bstatic -lc++ -Wl,-Bdynamic
    endif
else
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
endif

SRC := $(wildcard *.cpp)
EXE := $(SRC:.cpp=)

print-% : ; @echo "$* = $($*)"

.PHONY: $(SRC)

all: $(EXE)

ifeq ($(OSPRETTY), macOS)
%: %.cpp
	$(CXX) -arch arm64 -o $@_arm64 $< $(CXXFLAGS) $(INCL) $(LDFLAGS) \
	    $(LDLIBS)
	$(CXX) -arch x86_64 -o $@_x86_64 $< $(CXXFLAGS) $(INCL) $(LDFLAGS) \
	    $(LDLIBS)
	lipo -create -output $@ $@_arm64 $@_x86_64
endif

ifeq ($(OSPRETTY), macOS)
clean:
	$(RM) $(EXE) $(foreach n,$(EXE),$n_arm64 $n_x86_64)
else
clean:
	$(RM) $(EXE)
endif
