+++
title = 'Using a Single Header C Library in Zig'
date = 2023-10-14T10:08:58+03:00
draft = false
+++
I tried using raygui.h in Zig but ran into multiple build issues. The key was something that was discussed in multiple Zig discussions and issues. 
Currently, you need to wrap the .h file in a .c file like this. I call this raygui_impl.c like others suggested in the repo for Zig. There is discussion about this in here: 
[I found this here.](https://github.com/ziglang/zig/issues/17302#issuecomment-1737417445)

    #define RAYGUI_IMPLEMENTATION
	#include "raygui.h"  

Then, in my case the header file in declared as a dependency alongside raylib in my build.zig.zon file like this (I'm using my own fork of raylib, since when using Zig nightly, the normal raylib build.zig file is not working, I wrote a post about that before this one). So here is my build.zig.zon file:

    
	.{
	    .name = "my-game",
	    .version = "0.0.1",
	    .paths = .{""},
	    .dependencies = .{
	        .raylib = .{
	            .url = "https://github.com/Katajisto/raylib-zig-nightly/archive/refs/heads/master.tar.gz", // using a fork for zig nightly correct build.zig
	            .hash = "12203c3836e3f3f987c013b74ec62afe9a7c5790910eedff144c0352c3362f53d800",
	        },
	        .raygui = .{
	            .url = "https://github.com/raysan5/raygui/archive/refs/tags/4.0.tar.gz",
	            .hash = "1220b64eaeaf55609b3152b64d108ea73981c9689e783bddc68c80e77d275ebac164"
	        }
	    },
	}

Then, in my build.zig, I include and link it to the package.

    const rg = b.dependency("raygui", .{
        .target = target,
        .optimize = optimize,
    });
    const includepath = rg.path("/src/");
    exe.addIncludePath(includepath);
    exe.addCSourceFiles(.{ .files = &[_][]const u8{"./src/c/raygui_impl.c"}, .flags = &[_][]const u8{ "-g", "-O3" } });

This means that I have the raygui.h in my include path, so the c file has access to it and I also add the C source file. 
