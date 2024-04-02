# NixMLShell
A simple nix-shell that makes it mucho bueno to set up some popular python ml libraries, as well as any other pip packages not available in nixpkgs.

I stuggled quite a bit trying to wrap my head around the "nix" way of development. Turns out, it's kinda a pain. I love it.

This is a nix shell that sets up openGL / some other standard libraries to work with opencv/pytorch/numpy, and creates a python virtual environment to make it easy to install them with pip.

I will add that this is not the "nix" way of doing things - a better way would be using a pip requirements file or using the nix packages repository, but aint no body got time for that.

Make sure you have opengl enabled in your ```configuration.nix```:
```
hardware.opengl = {
  enable = true;
  driSupport = true;
  driSupport32Bit = true;
};
```

and paste the following lines into ```shell.nix``` alongside your project:
``` nix
{ pkgs ? import <nixpkgs> { } }:
with pkgs; mkShell {
  name = "ml-friendly-shell";
  buildInputs = [
    glib
    libGL
    glibc
    zlib
    python39
  ];
  shellHook = ''
    # Setup python virtual environment
    python -m venv .venv
    source .venv/bin/activate

    # Standard environment
    export LD_LIBRARY_PATH="$LD_LIBRARY_PATH:${pkgs.stdenv.cc.cc.lib}/lib"

    # [CV2] Software / Hardware bridge
    export LD_LIBRARY_PATH="$LD_LIBRARY_PATH:${libGL}/lib"

    # [CV2] Useful data types
    export LD_LIBRARY_PATH="$LD_LIBRARY_PATH:${pkgs.glib.out}/lib"

    # [Torch (CUDA)] OpenGL
    export LD_LIBRARY_PATH="$LD_LIBRARY_PATH:/run/opengl-driver/lib"
    export LD_LIBRARY_PATH="$LD_LIBRARY_PATH:/run/opengl-driver-32/lib"

    # [Numpy] Data compression
    export LD_LIBRARY_PATH="$LD_LIBRARY_PATH:${pkgs.zlib.outPath}/lib"
  '';
}
```

Then run:
```
nix-shell
```

Aftwards you can install your favourite packages with pip:
```
pip install torch opencv-python numpy
```

As a bonus tip, you can change the lines:
```
python -m venv .venv
source .venv/bin/activate
```
to
```
python -m venv ~/.venv
source ~/.venv/bin/activate
```
For a "global" python package install to not have to reinstall pytorch (~5gb!) for each project. 
