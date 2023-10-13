+++
title = 'Fix Raylib not building on nightly version of Zig'
date = 2023-10-13T23:54:50+03:00
draft = false
+++

I ran to this situation just a few moments ago. The C library Raylib stopped working on my
nightly Zig compiler. The issue is that Zig changed the parameters for the addCSourceFiles
method. It's weird that a new version of Zig would break the Raylib c library, but it's cause
there is a build.zig file in Raylib.

I fixed the build file in this fork of Raylib I made. [You can get it here](https://github.com/Katajisto/raylib-zig-nightly). I will try to keep that up to date with the raylib repository, but you can also be independent and just copy my changes and make a new fork of raylib.

If you know a better way for fixing the build file than a fork, please tell me.

You used to get an error like this when building.

    src/build.zig:23:11: error: member function expected 1 argument(s), found 2
    raylib.addCSourceFiles(&.{
This is because these functions don't take 2 params anymore:

    raylib.addCSourceFiles(&.{
        srcdir ++ "/rcore.c",
        srcdir ++ "/utils.c",
    }, raylib_flags);

They take in 1 parameter that is a struct like this:

    raylib.addCSourceFiles(.{
        .files = &.{
            srcdir ++ "/rcore.c",
            srcdir ++ "/utils.c",
        }, 
        .flags = raylib_flags 
    });

This is probably very obvious for most Zig users, but I just wanted to share my fork and this post in case someone else is trying to learn raylib and zig at the same time and is confused.