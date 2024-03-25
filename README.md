# NixMLShell
A simple nix-shell that makes it mucho bueno to set up pytorch and opencv, as well as any other pip packages.

I stuggled quite a bit trying to wrap my head around the "nix" way of development. Turns out, it's kinda a pain. I love it.

This is a nix shell that sets up openGL / some other standard libraries to work with opencv & pytorch, and creates a python virtual environment to make it easy to install them with pip.

Please note that I only use nvidia, and that i'm still learning and maybe there is a better way to do this.

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
  '';
}
```

Then run:
```
nix-shell
```

Aftwards you can install your favourite packages with pip:
```
pip install torch opencv-python
```
