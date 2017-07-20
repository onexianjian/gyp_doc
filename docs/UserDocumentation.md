<!DOCTYPE html><html lang="en"><head><meta charset="utf-8"><title>GYP - User Documentation</title><link rel="stylesheet" type="text/css" href="../+static/base.HLL9TqKl0YYybSzmT_wTdw.cache.css"/><link rel="stylesheet" type="text/css" href="../+static/doc.MXicahAOcEleYTBJrPykiw.cache.css"/><link rel="stylesheet" type="text/css" href="../+static/prettify/prettify.pZ5FqzM6cPxAflH0va2Ucw.cache.css"/></head><body class="Site"><header class="Site-header Site-header--withNavbar"><div class="Header"><div class="Header-title"><a class="Header-anchor" href="../index.md"><span class="Header-anchorTitle">GYP</span></a></div></div><nav class="Header-nav" role="navigation"><ul><li><a href="../index.md">Home</a></li><li><a href="UserDocumentation.md">User documentation</a></li><li><a href="InputFormatReference.md">Input Format Reference</a></li><li><a href="LanguageSpecification.md">Language specification</a></li><li><a href="Hacking.md">Hacking</a></li><li><a href="Testing.md">Testing</a></li><li><a href="GypVsCMake.md">GYP vs. CMake</a></li></ul></div></header><div class="Site-content Site-Content--markdown"><div class="Container"><div class="doc"><h1><a class="h" name="User-Documentation" href="UserDocumentation.md#User-Documentation"><span></span></a>User Documentation</h1><div class="toc" role="navigation"><h2>Contents</h2><div class="toc-aux"><ul><li><a href="UserDocumentation.md#Introduction">Introduction</a></li><li><a href="UserDocumentation.md#Skeleton-of-a-typical-Chromium-gyp-file">Skeleton of a typical Chromium .gyp file</a></li><li><a href="UserDocumentation.md#Skeleton-of-a-typical-executable-target-in-a-gyp-file">Skeleton of a typical executable target in a .gyp file</a></li><li><a href="UserDocumentation.md#Skeleton-of-a-typical-library-target-in-a-gyp-file">Skeleton of a typical library target in a .gyp file</a></li><li><a href="UserDocumentation.md#Use-Cases">Use Cases</a></li><ul><li><a href="UserDocumentation.md#Add-new-source-files">Add new source files</a></li><li><a href="UserDocumentation.md#Add-a-new-executable">Add a new executable</a></li><li><a href="UserDocumentation.md#Add-settings-to-a-target">Add settings to a target</a></li><li><a href="UserDocumentation.md#Cross_compiling">Cross-compiling</a></li><li><a href="UserDocumentation.md#Add-a-new-library">Add a new library</a></li><li><a href="UserDocumentation.md#Dependencies-between-targets">Dependencies between targets</a></li><li><a href="UserDocumentation.md#Support-for-Mac-OS-X-bundles">Support for Mac OS X bundles</a></li><li><a href="UserDocumentation.md#Move-files-refactoring">Move files (refactoring)</a></li><li><a href="UserDocumentation.md#Custom-build-steps">Custom build steps</a></li><li><a href="UserDocumentation.md#Build-flavors">Build flavors</a></li></ul></ul></div></div><h2><a class="h" name="Introduction" href="UserDocumentation.md#Introduction"><span></span></a>Introduction</h2><p>This document is intended to provide a user-level guide to GYP.  The emphasis here is on how to use GYP to accomplish specific tasks, not on the complete technical language specification.  (For that, see the <a href="LanguageSpecification.md">LanguageSpecification</a>.)</p><p>The document below starts with some overviews to provide context: an overview of the structure of a <code class="code">.gyp</code> file itself, an overview of a typical executable-program target in a <code class="code">.gyp</code> file, an an overview of a typical library target in a <code class="code">.gyp</code> file.</p><p>After the overviews, there are examples of <code class="code">gyp</code> patterns for different common use cases.</p><h2><a class="h" name="Skeleton-of-a-typical-Chromium-gyp-file" href="UserDocumentation.md#Skeleton-of-a-typical-Chromium-gyp-file"><span></span></a>Skeleton of a typical Chromium .gyp file</h2><p>Here is the skeleton of a typical <code class="code">.gyp</code> file in the Chromium tree:</p><pre class="code">  {
    &#39;variables&#39;: {
      .
      .
      .
    },
    &#39;includes&#39;: [
      &#39;../build/common.gypi&#39;,
    ],
    &#39;target_defaults&#39;: {
      .
      .
      .
    },
    &#39;targets&#39;: [
      {
        &#39;target_name&#39;: &#39;target_1&#39;,
          .
          .
          .
      },
      {
        &#39;target_name&#39;: &#39;target_2&#39;,
          .
          .
          .
      },
    ],
    &#39;conditions&#39;: [
      [&#39;OS==&quot;linux&quot;&#39;, {
        &#39;targets&#39;: [
          {
            &#39;target_name&#39;: &#39;linux_target_3&#39;,
              .
              .
              .
          },
        ],
      }],
      [&#39;OS==&quot;win&quot;&#39;, {
        &#39;targets&#39;: [
          {
            &#39;target_name&#39;: &#39;windows_target_4&#39;,
              .
              .
              .
          },
        ],
      }, { # OS != &quot;win&quot;
        &#39;targets&#39;: [
          {
            &#39;target_name&#39;: &#39;non_windows_target_5&#39;,
              .
              .
              .
          },
      }],
    ],
  }
</pre><p>The entire file just contains a Python dictionary.  (It&#39;s actually JSON, with two small Pythonic deviations: comments are introduced with <code class="code">#</code>, and a <code class="code">,</code> (comma)) is legal after the last element in a list or dictionary.)</p><p>The top-level pieces in the <code class="code">.gyp</code> file are as follows:</p><p><code class="code">&#39;variables&#39;</code>:  Definitions of variables that can be interpolated and used in various other parts of the file.</p><p><code class="code">&#39;includes&#39;</code>:  A list of of other files that will be included in this file.  By convention, included files have the suffix <code class="code">.gypi</code> (gyp include).</p><p><code class="code">&#39;target_defaults&#39;</code>:  Settings that will apply to <em>all</em> of the targets defined in this <code class="code">.gyp</code> file.</p><p><code class="code">&#39;targets&#39;</code>:  The list of targets for which this <code class="code">.gyp</code> file can generate builds.  Each target is a dictionary that contains settings describing all the information necessary to build the target.</p><p><code class="code">&#39;conditions&#39;</code>:  A list of condition specifications that can modify the contents of the items in the global dictionary defined by this <code class="code">.gyp</code> file based on the values of different variablwes.  As implied by the above example, the most common use of a <code class="code">conditions</code> section in the top-level dictionary is to add platform-specific targets to the <code class="code">targets</code> list.</p><h2><a class="h" name="Skeleton-of-a-typical-executable-target-in-a-gyp-file" href="UserDocumentation.md#Skeleton-of-a-typical-executable-target-in-a-gyp-file"><span></span></a>Skeleton of a typical executable target in a .gyp file</h2><p>The most straightforward target is probably a simple executable program. Here is an example <code class="code">executable</code> target that demonstrates the features that should cover most simple uses of gyp:</p><pre class="code">  {
    &#39;targets&#39;: [
      {
        &#39;target_name&#39;: &#39;foo&#39;,
        &#39;type&#39;: &#39;executable&#39;,
        &#39;msvs_guid&#39;: &#39;5ECEC9E5-8F23-47B6-93E0-C3B328B3BE65&#39;,
        &#39;dependencies&#39;: [
          &#39;xyzzy&#39;,
          &#39;../bar/bar.gyp:bar&#39;,
        ],
        &#39;defines&#39;: [
          &#39;DEFINE_FOO&#39;,
          &#39;DEFINE_A_VALUE=value&#39;,
        ],
        &#39;include_dirs&#39;: [
          &#39;..&#39;,
        ],
        &#39;sources&#39;: [
          &#39;file1.cc&#39;,
          &#39;file2.cc&#39;,
        ],
        &#39;conditions&#39;: [
          [&#39;OS==&quot;linux&quot;&#39;, {
            &#39;defines&#39;: [
              &#39;LINUX_DEFINE&#39;,
            ],
            &#39;include_dirs&#39;: [
              &#39;include/linux&#39;,
            ],
          }],
          [&#39;OS==&quot;win&quot;&#39;, {
            &#39;defines&#39;: [
              &#39;WINDOWS_SPECIFIC_DEFINE&#39;,
            ],
          }, { # OS != &quot;win&quot;,
            &#39;defines&#39;: [
              &#39;NON_WINDOWS_DEFINE&#39;,
            ],
          }]
        ],
      },
    ],
  }
</pre><p>The top-level settings in the target include:</p><p><code class="code">&#39;target_name&#39;</code>: The name by which the target should be known, which should be unique across all <code class="code">.gyp</code> files.  This name will be used as the project name in the generated Visual Studio solution, as the target name in the generated XCode configuration, and as the alias for building this target from the command line of the generated SCons configuration.</p><p><code class="code">&#39;type&#39;</code>: Set to <code class="code">executable</code>, logically enough.</p><p><code class="code">&#39;msvs_guid&#39;</code>: THIS IS ONLY TRANSITIONAL.  This is a hard-coded GUID values that will be used in the generated Visual Studio solution file(s).  This allows us to check in a <code class="code">chrome.sln</code> file that interoperates with gyp-generated project files.  Once everything in Chromium is being generated by gyp, it will no longer be important that the GUIDs stay constant across invocations, and we&#39;ll likely get rid of these settings,</p><p><code class="code">&#39;dependencies&#39;</code>: This lists other targets that this target depends on. The gyp-generated files will guarantee that the other targets are built before this target.  Any library targets in the <code class="code">dependencies</code> list will be linked with this target.  The various settings (<code class="code">defines</code>, <code class="code">include_dirs</code>, etc.) listed in the <code class="code">direct_dependent_settings</code> sections of the targets in this list will be applied to how <em>this</em> target is built and linked.  See the more complete discussion of <code class="code">direct_dependent_settings</code>, below.</p><p><code class="code">&#39;defines&#39;</code>: The C preprocessor definitions that will be passed in on compilation command lines (using <code class="code">-D</code> or <code class="code">/D</code> options).</p><p><code class="code">&#39;include_dirs&#39;</code>: The directories in which included header files live. These will be passed in on compilation command lines (using <code class="code">-I</code> or <code class="code">/I</code> options).</p><p><code class="code">&#39;sources&#39;</code>: The source files for this target.</p><p><code class="code">&#39;conditions&#39;</code>: A block of conditions that will be evaluated to update the different settings in the target dictionary.</p><h2><a class="h" name="Skeleton-of-a-typical-library-target-in-a-gyp-file" href="UserDocumentation.md#Skeleton-of-a-typical-library-target-in-a-gyp-file"><span></span></a>Skeleton of a typical library target in a .gyp file</h2><p>The vast majority of targets are libraries.  Here is an example of a library target including the additional features that should cover most needs of libraries:</p><pre class="code">  {
    &#39;targets&#39;: [
      {
        &#39;target_name&#39;: &#39;foo&#39;,
        &#39;type&#39;: &#39;&lt;(library)&#39;
        &#39;msvs_guid&#39;: &#39;5ECEC9E5-8F23-47B6-93E0-C3B328B3BE65&#39;,
        &#39;dependencies&#39;: [
          &#39;xyzzy&#39;,
          &#39;../bar/bar.gyp:bar&#39;,
        ],
        &#39;defines&#39;: [
          &#39;DEFINE_FOO&#39;,
          &#39;DEFINE_A_VALUE=value&#39;,
        ],
        &#39;include_dirs&#39;: [
          &#39;..&#39;,
        ],
        &#39;direct_dependent_settings&#39;: {
          &#39;defines&#39;: [
            &#39;DEFINE_FOO&#39;,
            &#39;DEFINE_ADDITIONAL&#39;,
          ],
          &#39;linkflags&#39;: [
          ],
        },
        &#39;export_dependent_settings&#39;: [
          &#39;../bar/bar.gyp:bar&#39;,
        ],
        &#39;sources&#39;: [
          &#39;file1.cc&#39;,
          &#39;file2.cc&#39;,
        ],
        &#39;conditions&#39;: [
          [&#39;OS==&quot;linux&quot;&#39;, {
            &#39;defines&#39;: [
              &#39;LINUX_DEFINE&#39;,
            ],
            &#39;include_dirs&#39;: [
              &#39;include/linux&#39;,
            ],
          ],
          [&#39;OS==&quot;win&quot;&#39;, {
            &#39;defines&#39;: [
              &#39;WINDOWS_SPECIFIC_DEFINE&#39;,
            ],
          }, { # OS != &quot;win&quot;,
            &#39;defines&#39;: [
              &#39;NON_WINDOWS_DEFINE&#39;,
            ],
          }]
        ],
    ],
  }
</pre><p>The possible entries in a library target are largely the same as those that can be specified for an executable target (<code class="code">defines</code>, <code class="code">include_dirs</code>, etc.).  The differences include:</p><p><code class="code">&#39;type&#39;</code>: This should almost always be set to &lsquo;&lt;(library)&rsquo;, which allows the user to define at gyp time whether libraries are to be built static or shared.  (On Linux, at least, linking with shared libraries saves significant link time.) If it&#39;s necessary to pin down the type of library to be built, the <code class="code">type</code> can be set explicitly to <code class="code">static_library</code> or <code class="code">shared_library</code>.</p><p><code class="code">&#39;direct_dependent_settings&#39;</code>: This defines the settings that will be applied to other targets that <em>directly depend</em> on this target--that is, that list <em>this</em> target in their <code class="code">&#39;dependencies&#39;</code> setting.  This is where you list the <code class="code">defines</code>, <code class="code">include_dirs</code>, <code class="code">cflags</code> and <code class="code">linkflags</code> that other targets that compile or link against this target need to build consistently.</p><p><code class="code">&#39;export_dependent_settings&#39;</code>: This lists the targets whose <code class="code">direct_dependent_settings</code> should be &ldquo;passed on&rdquo; to other targets that use (depend on) this target.  <code class="code">TODO: expand on this description.</code></p><h2><a class="h" name="Use-Cases" href="UserDocumentation.md#Use-Cases"><span></span></a>Use Cases</h2><p>These use cases are intended to cover the most common actions performed by developers using GYP.</p><p>Note that these examples are <em>not</em> fully-functioning, self-contained examples (or else they&lsquo;d be way too long).  Each example mostly contains just the keywords and settings relevant to the example, with perhaps a few extra keywords for context.  The intent is to try to show the specific pieces you need to pay attention to when doing something. [NOTE:  if practical use shows that these examples are confusing without additional context, please add what&rsquo;s necessary to clarify things.]</p><h3><a class="h" name="Add-new-source-files" href="UserDocumentation.md#Add-new-source-files"><span></span></a>Add new source files</h3><p>There are similar but slightly different patterns for adding a platform-independent source file vs. adding a source file that only builds on some of the supported platforms.</p><h4><a class="h" name="Add-a-source-file-that-builds-on-all-platforms" href="UserDocumentation.md#Add-a-source-file-that-builds-on-all-platforms"><span></span></a>Add a source file that builds on all platforms</h4><p><strong>Simplest possible case</strong>: You are adding a file(s) that builds on all platforms.</p><p>Just add the file(s) to the <code class="code">sources</code> list of the appropriate dictionary in the <code class="code">targets</code> list:</p><pre class="code">  {
    &#39;targets&#39;: [
      {
        &#39;target_name&#39;: &#39;my_target&#39;,
        &#39;type&#39;: &#39;executable&#39;,
        &#39;sources&#39;: [
          &#39;../other/file_1.cc&#39;,
          &#39;new_file.cc&#39;,
          &#39;subdir/file3.cc&#39;,
        ],
      },
    ],
  },
</pre><p>File path names are relative to the directory in which the <code class="code">.gyp</code> file lives.</p><p>Keep the list sorted alphabetically (unless there&#39;s a really, really, <em>really</em> good reason not to).</p><h4><a class="h" name="Add-a-platform_specific-source-file" href="UserDocumentation.md#Add-a-platform_specific-source-file"><span></span></a>Add a platform-specific source file</h4><h5><a class="h" name="Your-platform_specific-file-is-named-or" href="UserDocumentation.md#Your-platform_specific-file-is-named-or"><span></span></a>Your platform-specific file is named <code class="code">*_linux.{ext}</code>, <code class="code">*_mac.{ext}</code>, <code class="code">*_posix.{ext}</code> or <code class="code">*_win.{ext}</code></h5><p>The simplest way to add a platform-specific source file, assuming you&#39;re adding a completely new file and get to name it, is to use one of the following standard suffixes:</p><ul><li><code class="code">_linux</code>  (e.g. <code class="code">foo_linux.cc</code>)</li><li><code class="code">_mac</code>    (e.g. <code class="code">foo_mac.cc</code>)</li><li><code class="code">_posix</code>  (e.g. <code class="code">foo_posix.cc</code>)</li><li><code class="code">_win</code>    (e.g. <code class="code">foo_win.cc</code>)</li></ul><p>Simply add the file to the <code class="code">sources</code> list of the appropriate dict within the <code class="code">targets</code> list, like you would any other source file.</p><pre class="code">  {
    &#39;targets&#39;: [
      {
        &#39;target_name&#39;: &#39;foo&#39;,
        &#39;type&#39;: &#39;executable&#39;,
        &#39;sources&#39;: [
          &#39;independent.cc&#39;,
          &#39;specific_win.cc&#39;,
        ],
      },
    ],
  },
</pre><p>The Chromium <code class="code">.gyp</code> files all have appropriate <code class="code">conditions</code> entries to filter out the files that aren&#39;t appropriate for the current platform. In the above example, the <code class="code">specific_win.cc</code> file will be removed automatically from the source-list on non-Windows builds.</p><h5><a class="h" name="Your-platform_specific-file-does-not-use-an-already_defined-pattern" href="UserDocumentation.md#Your-platform_specific-file-does-not-use-an-already_defined-pattern"><span></span></a>Your platform-specific file does not use an already-defined pattern</h5><p>If your platform-specific file does not contain a <code class="code">*_{linux,mac,posix,win}</code> substring (or some other pattern that&lsquo;s already in the <code class="code">conditions</code> for the target), and you can&rsquo;t change the file name, there are two patterns that can be used.</p><p><strong>Prefererred</strong>:  Add the file to the <code class="code">sources</code> list of the appropriate dictionary within the <code class="code">targets</code> list.  Add an appropriate <code class="code">conditions</code> section to exclude the specific files name:</p><pre class="code">  {
    &#39;targets&#39;: [
      {
        &#39;target_name&#39;: &#39;foo&#39;,
        &#39;type&#39;: &#39;executable&#39;,
        &#39;sources&#39;: [
          &#39;linux_specific.cc&#39;,
        ],
        &#39;conditions&#39;: [
          [&#39;OS != &quot;linux&quot;&#39;, {
            &#39;sources!&#39;: [
              # Linux-only; exclude on other platforms.
              &#39;linux_specific.cc&#39;,
            ]
          }[,
        ],
      },
    ],
  },
</pre><p>Despite the duplicate listing, the above is generally preferred because the <code class="code">sources</code> list contains a useful global list of all sources on all platforms with consistent sorting on all platforms.</p><p><strong>Non-preferred</strong>: In some situations, however, it might make sense to list a platform-specific file only in a <code class="code">conditions</code> section that specifically <em>includes</em> it in the <code class="code">sources</code> list:</p><pre class="code">  {
    &#39;targets&#39;: [
      {
        &#39;target_name&#39;: &#39;foo&#39;,
        &#39;type&#39;: &#39;executable&#39;,
        &#39;sources&#39;: [],
        [&#39;OS == &quot;linux&quot;&#39;, {
          &#39;sources&#39;: [
            # Only add to sources list on Linux.
            &#39;linux_specific.cc&#39;,
          ]
        }],
      },
    ],
  },
</pre><p>The above two examples end up generating equivalent builds, with the small exception that the <code class="code">sources</code> lists will list the files in different orders.  (The first example defines explicitly where <code class="code">linux_specific.cc</code> appears in the list--perhaps in in the middle--whereas the second example will always tack it on to the end of the list.)</p><p><strong>Including or excluding files using patterns</strong>: There are more complicated ways to construct a <code class="code">sources</code> list based on patterns.  See <code class="code">TODO</code> below.</p><h3><a class="h" name="Add-a-new-executable" href="UserDocumentation.md#Add-a-new-executable"><span></span></a>Add a new executable</h3><p>An executable program is probably the most straightforward type of target, since all it typically needs is a list of source files, some compiler/linker settings (probably varied by platform), and some library targets on which it depends and which must be used in the final link.</p><h4><a class="h" name="Add-an-executable-that-builds-on-all-platforms" href="UserDocumentation.md#Add-an-executable-that-builds-on-all-platforms"><span></span></a>Add an executable that builds on all platforms</h4><p>Add a dictionary defining the new executable target to the <code class="code">targets</code> list in the appropriate <code class="code">.gyp</code> file.  Example:</p><pre class="code">  {
    &#39;targets&#39;: [
      {
        &#39;target_name&#39;: &#39;new_unit_tests&#39;,
        &#39;type&#39;: &#39;executable&#39;,
        &#39;defines&#39;: [
          &#39;FOO&#39;,
        ],
        &#39;include_dirs&#39;: [
          &#39;..&#39;,
        ],
        &#39;dependencies&#39;: [
          &#39;other_target_in_this_file&#39;,
          &#39;other_gyp2:target_in_other_gyp2&#39;,
        ],
        &#39;sources&#39;: [
          &#39;new_additional_source.cc&#39;,
          &#39;new_unit_tests.cc&#39;,
        ],
      },
    ],
  }
</pre><h4><a class="h" name="Add-a-platform_specific-executable" href="UserDocumentation.md#Add-a-platform_specific-executable"><span></span></a>Add a platform-specific executable</h4><p>Add a dictionary defining the new executable target to the <code class="code">targets</code> list within an appropriate <code class="code">conditions</code> block for the platform.  The <code class="code">conditions</code> block should be a sibling to the top-level <code class="code">targets</code> list:</p><pre class="code">  {
    &#39;targets&#39;: [
    ],
    &#39;conditions&#39;: [
      [&#39;OS==&quot;win&quot;&#39;, {
        &#39;targets&#39;: [
          {
            &#39;target_name&#39;: &#39;new_unit_tests&#39;,
            &#39;type&#39;: &#39;executable&#39;,
            &#39;defines&#39;: [
              &#39;FOO&#39;,
            ],
            &#39;include_dirs&#39;: [
              &#39;..&#39;,
            ],
            &#39;dependencies&#39;: [
              &#39;other_target_in_this_file&#39;,
              &#39;other_gyp2:target_in_other_gyp2&#39;,
            ],
            &#39;sources&#39;: [
              &#39;new_additional_source.cc&#39;,
              &#39;new_unit_tests.cc&#39;,
            ],
          },
        ],
      }],
    ],
  }
</pre><h3><a class="h" name="Add-settings-to-a-target" href="UserDocumentation.md#Add-settings-to-a-target"><span></span></a>Add settings to a target</h3><p>There are several different types of settings that can be defined for any given target.</p><h4><a class="h" name="Add-new-preprocessor-definitions-or-flags" href="UserDocumentation.md#Add-new-preprocessor-definitions-or-flags"><span></span></a>Add new preprocessor definitions (<code class="code">-D</code> or <code class="code">/D</code> flags)</h4><p>New preprocessor definitions are added by the <code class="code">defines</code> setting:</p><pre class="code">  {
    &#39;targets&#39;: [
      {
        &#39;target_name&#39;: &#39;existing_target&#39;,
        &#39;defines&#39;: [
          &#39;FOO&#39;,
          &#39;BAR=some_value&#39;,
        ],
      },
    ],
  },
</pre><p>These may be specified directly in a target&#39;s settings, as in the above example, or in a <code class="code">conditions</code> section.</p><h4><a class="h" name="Add-a-new-include-directory-or-flags" href="UserDocumentation.md#Add-a-new-include-directory-or-flags"><span></span></a>Add a new include directory (<code class="code">-I</code> or <code class="code">/I</code> flags)</h4><p>New include directories are added by the <code class="code">include_dirs</code> setting:</p><pre class="code">  {
    &#39;targets&#39;: [
      {
        &#39;target_name&#39;: &#39;existing_target&#39;,
        &#39;include_dirs&#39;: [
          &#39;..&#39;,
          &#39;include&#39;,
        ],
      },
    ],
  },
</pre><p>These may be specified directly in a target&#39;s settings, as in the above example, or in a <code class="code">conditions</code> section.</p><h4><a class="h" name="Add-new-compiler-flags" href="UserDocumentation.md#Add-new-compiler-flags"><span></span></a>Add new compiler flags</h4><p>Specific compiler flags can be added with the <code class="code">cflags</code> setting:</p><pre class="code">  {
    &#39;targets&#39;: [
      {
        &#39;target_name&#39;: &#39;existing_target&#39;,
        &#39;conditions&#39;: [
          [&#39;OS==&quot;win&quot;&#39;, {
            &#39;cflags&#39;: [
              &#39;/WX&#39;,
            ],
          }, { # OS != &quot;win&quot;
            &#39;cflags&#39;: [
              &#39;-Werror&#39;,
            ],
          }],
        ],
      },
    ],
  },
</pre><p>Because these flags will be specific to the actual compiler involved, they will almost always be only set within a <code class="code">conditions</code> section.</p><h4><a class="h" name="Add-new-linker-flags" href="UserDocumentation.md#Add-new-linker-flags"><span></span></a>Add new linker flags</h4><p>Setting linker flags is OS-specific. On linux and most non-mac posix systems, they can be added with the <code class="code">ldflags</code> setting:</p><pre class="code">  {
    &#39;targets&#39;: [
      {
        &#39;target_name&#39;: &#39;existing_target&#39;,
        &#39;conditions&#39;: [
          [&#39;OS==&quot;linux&quot;&#39;, {
            &#39;ldflags&#39;: [
              &#39;-pthread&#39;,
            ],
          }],
        ],
      },
    ],
  },
</pre><p>Because these flags will be specific to the actual linker involved, they will almost always be only set within a <code class="code">conditions</code> section.</p><p>On OS X, linker settings are set via <code class="code">xcode_settings</code>, on Windows via <code class="code">msvs_settings</code>.</p><h4><a class="h" name="Exclude-settings-on-a-platform" href="UserDocumentation.md#Exclude-settings-on-a-platform"><span></span></a>Exclude settings on a platform</h4><p>Any given settings keyword (<code class="code">defines</code>, <code class="code">include_dirs</code>, etc.) has a corresponding form with a trailing <code class="code">!</code> (exclamation point) to remove values from a setting.  One useful example of this is to remove the Linux <code class="code">-Werror</code> flag from the global settings defined in <code class="code">build/common.gypi</code>:</p><pre class="code">  {
    &#39;targets&#39;: [
      {
        &#39;target_name&#39;: &#39;third_party_target&#39;,
        &#39;conditions&#39;: [
          [&#39;OS==&quot;linux&quot;&#39;, {
            &#39;cflags!&#39;: [
              &#39;-Werror&#39;,
            ],
          }],
        ],
      },
    ],
  },
</pre><h3><a class="h" name="Cross_compiling" href="UserDocumentation.md#Cross_compiling"><span></span></a>Cross-compiling</h3><p>GYP has some (relatively limited) support for cross-compiling.</p><p>If the variable <code class="code">GYP_CROSSCOMPILE</code> or one of the toolchain-related variables (like <code class="code">CC_host</code> or <code class="code">CC_target</code>) is set, GYP will think that you wish to do a cross-compile.</p><p>When cross-compiling, each target can be part of a &ldquo;host&rdquo; build, a &ldquo;target&rdquo; build, or both. By default, the target is assumed to be (only) part of the &ldquo;target&rdquo; build. The &lsquo;toolsets&rsquo; property can be set on a target to change the default.</p><p>A target&#39;s dependencies are assumed to match the build type (so, if A depends on B, by default that means that a target build of A depends on a target build of B). You can explicitly depend on targets across toolchains by specifying &ldquo;#host&rdquo; or &ldquo;#target&rdquo; in the dependencies list. If GYP is not doing a cross-compile, the &ldquo;#host&rdquo; and &ldquo;#target&rdquo; will be stripped as needed, so nothing breaks.</p><h3><a class="h" name="Add-a-new-library" href="UserDocumentation.md#Add-a-new-library"><span></span></a>Add a new library</h3><p>TODO:  write intro</p><h4><a class="h" name="Add-a-library-that-builds-on-all-platforms" href="UserDocumentation.md#Add-a-library-that-builds-on-all-platforms"><span></span></a>Add a library that builds on all platforms</h4><p>Add the a dictionary defining the new library target to the <code class="code">targets</code> list in the appropriate <code class="code">.gyp</code> file.  Example:</p><pre class="code">  {
    &#39;targets&#39;: [
      {
        &#39;target_name&#39;: &#39;new_library&#39;,
        &#39;type&#39;: &#39;&lt;(library)&#39;,
        &#39;defines&#39;: [
          &#39;FOO&#39;,
          &#39;BAR=some_value&#39;,
        ],
        &#39;include_dirs&#39;: [
          &#39;..&#39;,
        ],
        &#39;dependencies&#39;: [
          &#39;other_target_in_this_file&#39;,
          &#39;other_gyp2:target_in_other_gyp2&#39;,
        ],
        &#39;direct_dependent_settings&#39;: {
          &#39;include_dirs&#39;: &#39;.&#39;,
        },
        &#39;export_dependent_settings&#39;: [
          &#39;other_target_in_this_file&#39;,
        ],
        &#39;sources&#39;: [
          &#39;new_additional_source.cc&#39;,
          &#39;new_library.cc&#39;,
        ],
      },
    ],
  }
</pre><p>The use of the <code class="code">&lt;(library)</code> variable above should be the default <code class="code">type</code> setting for most library targets, as it allows the developer to choose, at <code class="code">gyp</code> time, whether to build with static or shared libraries. (Building with shared libraries saves a <em>lot</em> of link time on Linux.)</p><p>It may be necessary to build a specific library as a fixed type.  Is so, the <code class="code">type</code> field can be hard-wired appropriately.  For a static library:</p><pre class="code">        &#39;type&#39;: &#39;static_library&#39;,
</pre><p>For a shared library:</p><pre class="code">        &#39;type&#39;: &#39;shared_library&#39;,
</pre><h4><a class="h" name="Add-a-platform_specific-library" href="UserDocumentation.md#Add-a-platform_specific-library"><span></span></a>Add a platform-specific library</h4><p>Add a dictionary defining the new library target to the <code class="code">targets</code> list within a <code class="code">conditions</code> block that&#39;s a sibling to the top-level <code class="code">targets</code> list:</p><pre class="code">  {
    &#39;targets&#39;: [
    ],
    &#39;conditions&#39;: [
      [&#39;OS==&quot;win&quot;&#39;, {
        &#39;targets&#39;: [
          {
            &#39;target_name&#39;: &#39;new_library&#39;,
            &#39;type&#39;: &#39;&lt;(library)&#39;,
            &#39;defines&#39;: [
              &#39;FOO&#39;,
              &#39;BAR=some_value&#39;,
            ],
            &#39;include_dirs&#39;: [
              &#39;..&#39;,
            ],
            &#39;dependencies&#39;: [
              &#39;other_target_in_this_file&#39;,
              &#39;other_gyp2:target_in_other_gyp2&#39;,
            ],
            &#39;direct_dependent_settings&#39;: {
              &#39;include_dirs&#39;: &#39;.&#39;,
            },
            &#39;export_dependent_settings&#39;: [
              &#39;other_target_in_this_file&#39;,
            ],
            &#39;sources&#39;: [
              &#39;new_additional_source.cc&#39;,
              &#39;new_library.cc&#39;,
            ],
          },
        ],
      }],
    ],
  }
</pre><h3><a class="h" name="Dependencies-between-targets" href="UserDocumentation.md#Dependencies-between-targets"><span></span></a>Dependencies between targets</h3><p>GYP provides useful primitives for establishing dependencies between targets, which need to be configured in the following situations.</p><h4><a class="h" name="Linking-with-another-library-target" href="UserDocumentation.md#Linking-with-another-library-target"><span></span></a>Linking with another library target</h4><pre class="code">  {
    &#39;targets&#39;: [
      {
        &#39;target_name&#39;: &#39;foo&#39;,
        &#39;dependencies&#39;: [
          &#39;libbar&#39;,
        ],
      },
      {
        &#39;target_name&#39;: &#39;libbar&#39;,
        &#39;type&#39;: &#39;&lt;(library)&#39;,
        &#39;sources&#39;: [
        ],
      },
    ],
  }
</pre><p>Note that if the library target is in a different <code class="code">.gyp</code> file, you have to specify the path to other <code class="code">.gyp</code> file, relative to this <code class="code">.gyp</code> file&#39;s directory:</p><pre class="code">  {
    &#39;targets&#39;: [
      {
        &#39;target_name&#39;: &#39;foo&#39;,
        &#39;dependencies&#39;: [
          &#39;../bar/bar.gyp:libbar&#39;,
        ],
      },
    ],
  }
</pre><p>Adding a library often involves updating multiple <code class="code">.gyp</code> files, adding the target to the approprate <code class="code">.gyp</code> file (possibly a newly-added <code class="code">.gyp</code> file), and updating targets in the other <code class="code">.gyp</code> files that depend on (link with) the new library.</p><h4><a class="h" name="Compiling-with-necessary-flags-for-a-library-target-dependency" href="UserDocumentation.md#Compiling-with-necessary-flags-for-a-library-target-dependency"><span></span></a>Compiling with necessary flags for a library target dependency</h4><p>We need to build a library (often a third-party library) with specific preprocessor definitions or command-line flags, and need to ensure that targets that depend on the library build with the same settings.  This situation is handled by a <code class="code">direct_dependent_settings</code> block:</p><pre class="code">  {
    &#39;targets&#39;: [
      {
        &#39;target_name&#39;: &#39;foo&#39;,
        &#39;type&#39;: &#39;executable&#39;,
        &#39;dependencies&#39;: [
          &#39;libbar&#39;,
        ],
      },
      {
        &#39;target_name&#39;: &#39;libbar&#39;,
        &#39;type&#39;: &#39;&lt;(library)&#39;,
        &#39;defines&#39;: [
          &#39;LOCAL_DEFINE_FOR_LIBBAR&#39;,
          &#39;DEFINE_TO_USE_LIBBAR&#39;,
        ],
        &#39;include_dirs&#39;: [
          &#39;..&#39;,
          &#39;include/libbar&#39;,
        ],
        &#39;direct_dependent_settings&#39;: {
          &#39;defines&#39;: [
            &#39;DEFINE_TO_USE_LIBBAR&#39;,
          ],
          &#39;include_dirs&#39;: [
            &#39;include/libbar&#39;,
          ],
        },
      },
    ],
  }
</pre><p>In the above example, the sources of the <code class="code">foo</code> executable will be compiled with the options <code class="code">-DDEFINE_TO_USE_LIBBAR -Iinclude/libbar</code>, because of those settings&#39; being listed in the <code class="code">direct_dependent_settings</code> block.</p><p>Note that these settings will likely need to be replicated in the settings for the library target itsef, so that the library will build with the same options.  This does not prevent the target from defining additional options for its &ldquo;internal&rdquo; use when compiling its own source files.  (In the above example, these are the <code class="code">LOCAL_DEFINE_FOR_LIBBAR</code> define, and the <code class="code">..</code> entry in the <code class="code">include_dirs</code> list.)</p><h4><a class="h" name="When-a-library-depends-on-an-additional-library-at-final-link-time" href="UserDocumentation.md#When-a-library-depends-on-an-additional-library-at-final-link-time"><span></span></a>When a library depends on an additional library at final link time</h4><pre class="code">  {
    &#39;targets&#39;: [
      {
        &#39;target_name&#39;: &#39;foo&#39;,
        &#39;type&#39;: &#39;executable&#39;,
        &#39;dependencies&#39;: [
          &#39;libbar&#39;,
        ],
      },
      {
        &#39;target_name&#39;: &#39;libbar&#39;,
        &#39;type&#39;: &#39;&lt;(library)&#39;,
        &#39;dependencies&#39;: [
          &#39;libother&#39;
        ],
        &#39;export_dependent_settings&#39;: [
          &#39;libother&#39;
        ],
      },
      {
        &#39;target_name&#39;: &#39;libother&#39;,
        &#39;type&#39;: &#39;&lt;(library)&#39;,
        &#39;direct_dependent_settings&#39;: {
          &#39;defines&#39;: [
            &#39;DEFINE_FOR_LIBOTHER&#39;,
          ],
          &#39;include_dirs&#39;: [
            &#39;include/libother&#39;,
          ],
        },
      },
    ],
  }
</pre><h3><a class="h" name="Support-for-Mac-OS-X-bundles" href="UserDocumentation.md#Support-for-Mac-OS-X-bundles"><span></span></a>Support for Mac OS X bundles</h3><p>gyp supports building bundles on OS X (.app, .framework, .bundle, etc). Here is an example of this:</p><pre class="code">    {
      &#39;target_name&#39;: &#39;test_app&#39;,
      &#39;product_name&#39;: &#39;Test App Gyp&#39;,
      &#39;type&#39;: &#39;executable&#39;,
      &#39;mac_bundle&#39;: 1,
      &#39;sources&#39;: [
        &#39;main.m&#39;,
        &#39;TestAppAppDelegate.h&#39;,
        &#39;TestAppAppDelegate.m&#39;,
      ],
      &#39;mac_bundle_resources&#39;: [
        &#39;TestApp/English.lproj/InfoPlist.strings&#39;,
        &#39;TestApp/English.lproj/MainMenu.xib&#39;,
      ],
      &#39;link_settings&#39;: {
        &#39;libraries&#39;: [
          &#39;$(SDKROOT)/System/Library/Frameworks/Cocoa.framework&#39;,
        ],
      },
      &#39;xcode_settings&#39;: {
        &#39;INFOPLIST_FILE&#39;: &#39;TestApp/TestApp-Info.plist&#39;,
      },
    },
</pre><p>The <code class="code">mac_bundle</code> key tells gyp that this target should be a bundle. <code class="code">executable</code> targets get extension <code class="code">.app</code> by default, <code class="code">shared_library</code> targets get <code class="code">.framework</code> – but you can change the bundle extensions by setting <code class="code">product_extension</code> if you want. Files listed in <code class="code">mac_bundle_resources</code> will be copied to the bundle&#39;s <code class="code">Resource</code> folder of the bundle. You can also set <code class="code">process_outputs_as_mac_bundle_resources</code> to 1 in actions and rules to let the output of actions and rules be added to that folder (similar to <code class="code">process_outputs_as_sources</code>). If <code class="code">product_name</code> is not set, the bundle will be named after <code class="code">target_name</code>as usual.</p><h3><a class="h" name="Move-files-refactoring" href="UserDocumentation.md#Move-files-refactoring"><span></span></a>Move files (refactoring)</h3><p>TODO</p><h3><a class="h" name="Custom-build-steps" href="UserDocumentation.md#Custom-build-steps"><span></span></a>Custom build steps</h3><p>TODO</p><h4><a class="h" name="Adding-an-explicit-build-step-to-generate-specific-files" href="UserDocumentation.md#Adding-an-explicit-build-step-to-generate-specific-files"><span></span></a>Adding an explicit build step to generate specific files</h4><p>TODO</p><h4><a class="h" name="Adding-a-rule-to-handle-files-with-a-new-suffix" href="UserDocumentation.md#Adding-a-rule-to-handle-files-with-a-new-suffix"><span></span></a>Adding a rule to handle files with a new suffix</h4><p>TODO</p><h3><a class="h" name="Build-flavors" href="UserDocumentation.md#Build-flavors"><span></span></a>Build flavors</h3><p>TODO</p></div></div></div><footer class="Site-footer"><div class="Footer"><div class="Footer-poweredBy">Powered by <a href="https://gerrit.googlesource.com/gitiles/">Gitiles</a></div><div class="Footer-links"></ul></div></footer></body></html>