<!DOCTYPE html><html lang="en"><head><meta charset="utf-8"><title>GYP - vs. CMake</title><link rel="stylesheet" type="text/css" href="../+static/base.HLL9TqKl0YYybSzmT_wTdw.cache.css"/><link rel="stylesheet" type="text/css" href="../+static/doc.MXicahAOcEleYTBJrPykiw.cache.css"/><link rel="stylesheet" type="text/css" href="../+static/prettify/prettify.pZ5FqzM6cPxAflH0va2Ucw.cache.css"/></head><body class="Site"><header class="Site-header Site-header--withNavbar"><div class="Header"><div class="Header-title"><a class="Header-anchor" href="../index.md"><span class="Header-anchorTitle">GYP</span></a></div></div><nav class="Header-nav" role="navigation"><ul><li><a href="../index.md">Home</a></li><li><a href="UserDocumentation.md">User documentation</a></li><li><a href="InputFormatReference.md">Input Format Reference</a></li><li><a href="LanguageSpecification.md">Language specification</a></li><li><a href="Hacking.md">Hacking</a></li><li><a href="Testing.md">Testing</a></li><li><a href="GypVsCMake.md">GYP vs. CMake</a></li></ul></div></header><div class="Site-content Site-Content--markdown"><div class="Container"><div class="doc"><h1><a class="h" name="vs_CMake" href="GypVsCMake.md#vs_CMake"><span></span></a>vs. CMake</h1><p>GYP was originally created to generate native IDE project files (Visual Studio, Xcode) for building <a href="http://www.chromim.org">Chromium</a>.</p><p>The functionality of GYP is very similar to the <a href="http://www.cmake.org">CMake</a> build tool.  Bradley Nelson wrote up the following description of why the team created GYP instead of using CMake.  The text below is copied from <a href="http://www.mail-archive.com/webkit-dev@lists.webkit.org/msg11029.html">http://www.mail-archive.com/webkit-dev@lists.webkit.org/msg11029.html</a></p><pre class="code"><br />Re: [webkit-dev] CMake as a build system?
Bradley Nelson
Mon, 19 Apr 2010 22:38:30 -0700

Here&#39;s the innards of an email with a laundry list of stuff I came up with a
while back on the gyp-developers list in response to Mike Craddick regarding
what motivated gyp&#39;s development, since we were aware of cmake at the time
(we&#39;d even started a speculative port):


I did an exploratory port of portions of Chromium to cmake (I think I got as
far as net, base, sandbox, and part of webkit).
There were a number of motivations, not all of which would apply to other
projects. Also, some of the design of gyp was informed by experience at
Google with large projects built wholly from source, leading to features
absent from cmake, but not strictly required for Chromium.

1. Ability to incrementally transition on Windows. It took us about 6 months
to switch fully to gyp. Previous attempts to move to scons had taken a long
time and failed, due to the requirement to transition while in flight. For a
substantial period of time, we had a hybrid of checked in vcproj and gyp generated
vcproj. To this day we still have a good number of GUIDs pinned in the gyp files,
because different parts of our release pipeline have leftover assumptions
regarding manipulating the raw sln/vcprojs. This transition occurred from
the bottom up, largely because modules like base were easier to convert, and
had a lower churn rate. During early stages of the transition, the majority
of the team wasn&#39;t even aware they were using gyp, as it integrated into
their existing workflow, and only affected modules that had been converted.

2. Generation of a more &#39;normal&#39; vcproj file. Gyp attempts, particularly on
Windows, to generate vcprojs which resemble hand generated projects. It
doesn&#39;t generate any Makefile type projects, but instead produces msvs
Custom Build Steps and Custom Build Rules. This makes the resulting projects
easier to understand from the IDE and avoids parts of the IDE that simply
don&#39;t function correctly if you use Makefile projects. Our early hope with
gyp was to support the least common denominator of features present in each
of the platform specific project file formats, rather than falling back on
generated Makefiles/shell scripts to emulate some common abstraction. CMake by
comparison makes a good faith attempt to use native project features, but
falls back on generated scripts in order to preserve the same semantics on
each platforms.

3. Abstraction on the level of project settings, rather than command line
flags. In gyp&#39;s syntax you can add nearly any option present in a hand
generated xcode/vcproj file. This allows you to use abstractions built into
the IDEs rather than reverse engineering them possibly incorrectly for
things like: manifest generation, precompiled headers, bundle generation.
When somebody wants to use a particular menu option from msvs, I&#39;m able to
do a web search on the name of the setting from the IDE and provide them
with a gyp stanza that does the equivalent. In many cases, not all project
file constructs correspond to command line flags.

4. Strong notion of module public/private interface. Gyp allows targets to
publish a set of direct_dependent_settings, specifying things like
include_dirs, defines, platforms specific settings, etc. This means that
when module A depends on module B, it automatically acquires the right build
settings without module A being filled with assumptions/knowledge of exactly
how module B is built. Additionally, all of the transitive dependencies of
module B are pulled in. This avoids their being a single top level view of
the project, rather each gyp file expresses knowledge about its immediate
neighbors. This keep local knowledge local. CMake effectively has a large
shared global namespace.

5. Cross platform generation. CMake is not able to generate all project
files on all platforms. For example xcode projects cannot be generated from
windows (cmake uses mac specific libraries to do project generation). This
means that for instance generating a tarball containing pregenerated
projects for all platforms is hard with Cmake (requires distribution to
several machine types).

6. Gyp has rudimentary cross compile support. Currently we&#39;ve added enough
functionality to gyp to support x86 -&gt; arm cross compiles. Last I checked
this functionality wasn&#39;t present in cmake. (This occurred later).


That being said there are a number of drawbacks currently to gyp:

1. Because platform specific settings are expressed at the project file
level (rather than the command line level). Settings which might otherwise
be shared in common between platforms (flags to gcc on mac/linux), end up
being repeated twice. Though in fairness there is actually less sharing here
than you&#39;d think. include_dirs and defines actually represent 90% of what
can be typically shared.

2. CMake may be more mature, having been applied to a broader range of
projects. There a number of &#39;tool modules&#39; for cmake, which are shared in a
common community.

3. gyp currently makes some nasty assumptions about the availability of
chromium&#39;s hermetic copy of cygwin on windows. This causes you to either
have to special case a number of rules, or swallow this copy of cygwin as a
build time dependency.

4. CMake includes a fairly readable imperative language. Currently Gyp has a
somewhat poorly specified declarative language (variable expansion happens
in sometimes weird and counter-intuitive ways). In fairness though, gyp assumes
that external python scripts can be used as an escape hatch. Also gyp avoids
a lot of the things you&#39;d need imperative code for, by having a nice target
settings publication mechanism.

5. (Feature/drawback depending on personal preference). Gyp&#39;s syntax is
DEEPLY nested. It suffers from all of Lisp&#39;s advantages and drawbacks.

-BradN
</pre></div></div></div><footer class="Site-footer"><div class="Footer"><div class="Footer-poweredBy">Powered by <a href="https://gerrit.googlesource.com/gitiles/">Gitiles</a></div><div class="Footer-links"></ul></div></footer></body></html>