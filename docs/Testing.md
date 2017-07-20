<!DOCTYPE html><html lang="en"><head><meta charset="utf-8"><title>GYP - Testing</title><link rel="stylesheet" type="text/css" href="../+static/base.HLL9TqKl0YYybSzmT_wTdw.cache.css"/><link rel="stylesheet" type="text/css" href="../+static/doc.MXicahAOcEleYTBJrPykiw.cache.css"/><link rel="stylesheet" type="text/css" href="../+static/prettify/prettify.pZ5FqzM6cPxAflH0va2Ucw.cache.css"/></head><body class="Site"><header class="Site-header Site-header--withNavbar"><div class="Header"><div class="Header-title"><a class="Header-anchor" href="../index.md"><span class="Header-anchorTitle">GYP</span></a></div></div><nav class="Header-nav" role="navigation"><ul><li><a href="../index.md">Home</a></li><li><a href="UserDocumentation.md">User documentation</a></li><li><a href="InputFormatReference.md">Input Format Reference</a></li><li><a href="LanguageSpecification.md">Language specification</a></li><li><a href="Hacking.md">Hacking</a></li><li><a href="Testing.md">Testing</a></li><li><a href="GypVsCMake.md">GYP vs. CMake</a></li></ul></div></header><div class="Site-content Site-Content--markdown"><div class="Container"><div class="doc"><h1><a class="h" name="Testing" href="Testing.md#Testing"><span></span></a>Testing</h1><div class="toc" role="navigation"><h2>Contents</h2><div class="toc-aux"><ul><li><a href="Testing.md#Testing">Testing</a></li><ul><li><a href="Testing.md#Introduction">Introduction</a></li><li><a href="Testing.md#Hello_world_GYP-test-configuration">Hello, world! GYP test configuration</a></li><li><a href="Testing.md#Running-tests">Running tests</a></li><li><a href="Testing.md#Debugging-tests">Debugging tests</a></li></ul><li><a href="Testing.md#Specifying-the-format-build-tool_to-use">Specifying the format (build tool) to use</a></li><ul><li><a href="Testing.md#Test-script-functions-and-methods">Test script functions and methods</a></li><ul><li><a href="Testing.md#Initialization">Initialization</a></li><li><a href="Testing.md#Running-GYP">Running GYP</a></li><li><a href="Testing.md#Running-the-build-tool">Running the build tool</a></li><li><a href="Testing.md#Running-executables">Running executables</a></li><li><a href="Testing.md#Fetching-command-output">Fetching command output</a></li><li><a href="Testing.md#Verifying-existence-or-non_existence-of-files-or-directories">Verifying existence or non-existence of files or directories</a></li><li><a href="Testing.md#Verifying-file-contents">Verifying file contents</a></li><li><a href="Testing.md#Reading-file-contents">Reading file contents</a></li><li><a href="Testing.md#Test-success-or-failure">Test success or failure</a></li></ul></ul></ul></div></div><h2><a class="h" name="Introduction" href="Testing.md#Introduction"><span></span></a>Introduction</h2><p>This document describes the GYP testing infrastructure, as provided by the <code class="code">TestGyp.py</code> module.</p><p>These tests emphasize testing the <em>behavior</em> of the various GYP-generated build configurations: Visual Studio, Xcode, SCons, Make, etc. The goal is <em>not</em> to test the output of the GYP generators by, for example, comparing a GYP-generated Makefile against a set of known &ldquo;golden&rdquo; Makefiles (although the testing infrastructure could be used to write those kinds of tests). The idea is that the generated build configuration files could be completely written to add a feature or fix a bug so long as they continue to support the functional behaviors defined by the tests:  building programs, shared libraries, etc.</p><h2><a class="h" name="Hello_world_GYP-test-configuration" href="Testing.md#Hello_world_GYP-test-configuration"><span></span></a>&ldquo;Hello, world!&rdquo; GYP test configuration</h2><p>Here is an actual test configuration, a simple build of a C program to print <code class="code">&quot;Hello, world!&quot;</code>.</p><pre class="code">  $ ls -l test/hello
  total 20
  -rw-r--r-- 1 knight knight 312 Jul 30 20:22 gyptest-all.py
  -rw-r--r-- 1 knight knight 307 Jul 30 20:22 gyptest-default.py
  -rwxr-xr-x 1 knight knight 326 Jul 30 20:22 gyptest-target.py
  -rw-r--r-- 1 knight knight  98 Jul 30 20:22 hello.c
  -rw-r--r-- 1 knight knight 142 Jul 30 20:22 hello.gyp
  $
</pre><p>The <code class="code">gyptest-*.py</code> files are three separate tests (test scripts) that use this configuration.  The first one, <code class="code">gyptest-all.py</code>, looks like this:</p><pre class="code">  #!/usr/bin/env python

  &quot;&quot;&quot;
  Verifies simplest-possible build of a &quot;Hello, world!&quot; program
  using an explicit build target of &#39;all&#39;.
  &quot;&quot;&quot;

  import TestGyp

  test = TestGyp.TestGyp()

  test.run_gyp(&#39;hello.gyp&#39;)

  test.build_all(&#39;hello.gyp&#39;)

  test.run_built_executable(&#39;hello&#39;, stdout=&quot;Hello, world!\n&quot;)

  test.pass_test()
</pre><p>The test script above runs GYP against the specified input file (<code class="code">hello.gyp</code>) to generate a build configuration. It then tries to build the <code class="code">&#39;all&#39;</code> target (or its equivalent) using the generated build configuration. Last, it verifies that the build worked as expected by running the executable program (<code class="code">hello</code>) that was just presumably built by the generated configuration, and verifies that the output from the program matches the expected <code class="code">stdout</code> string (<code class="code">&quot;Hello, world!\n&quot;</code>).</p><p>Which configuration is generated (i.e., which build tool to test) is specified when the test is run; see the next section.</p><p>Surrounding the functional parts of the test described above are the header, which should be basically the same for each test (modulo a different description in the docstring):</p><pre class="code">  #!/usr/bin/env python

  &quot;&quot;&quot;
  Verifies simplest-possible build of a &quot;Hello, world!&quot; program
  using an explicit build target of &#39;all&#39;.
  &quot;&quot;&quot;

  import TestGyp

  test = TestGyp.TestGyp()
</pre><p>Similarly, the footer should be the same in every test:</p><pre class="code">  test.pass_test()
</pre><h2><a class="h" name="Running-tests" href="Testing.md#Running-tests"><span></span></a>Running tests</h2><p>Test scripts are run by the <code class="code">gyptest.py</code> script. You can specify (an) explicit test script(s) to run:</p><pre class="code">  $ python gyptest.py test/hello/gyptest-all.py
  PYTHONPATH=/home/knight/src/gyp/trunk/test/lib
  TESTGYP_FORMAT=scons
  /usr/bin/python test/hello/gyptest-all.py
  PASSED
  $
</pre><p>If you specify a directory, all test scripts (scripts prefixed with <code class="code">gyptest-</code>) underneath the directory will be run:</p><pre class="code">  $ python gyptest.py test/hello
  PYTHONPATH=/home/knight/src/gyp/trunk/test/lib
  TESTGYP_FORMAT=scons
  /usr/bin/python test/hello/gyptest-all.py
  PASSED
  /usr/bin/python test/hello/gyptest-default.py
  PASSED
  /usr/bin/python test/hello/gyptest-target.py
  PASSED
  $
</pre><p>Or you can specify the <code class="code">-a</code> option to run all scripts in the tree:</p><pre class="code">  $ python gyptest.py -a
  PYTHONPATH=/home/knight/src/gyp/trunk/test/lib
  TESTGYP_FORMAT=scons
  /usr/bin/python test/configurations/gyptest-configurations.py
  PASSED
  /usr/bin/python test/defines/gyptest-defines.py
  PASSED
      .
      .
      .
      .
  /usr/bin/python test/variables/gyptest-commands.py
  PASSED
  $
</pre><p>If any tests fail during the run, the <code class="code">gyptest.py</code> script will report them in a summary at the end.</p><h2><a class="h" name="Debugging-tests" href="Testing.md#Debugging-tests"><span></span></a>Debugging tests</h2><p>Tests that create intermediate output do so under the gyp/out/testworkarea directory. On test completion, intermediate output is cleaned up. To preserve this output, set the environment variable PRESERVE=1. This can be handy to inspect intermediate data when debugging a test.</p><p>You can also set PRESERVE_PASS=1, PRESERVE_FAIL=1 or PRESERVE_NO_RESULT=1 to preserve output for tests that fall into one of those categories.</p><h1><a class="h" name="Specifying-the-format-build-tool_to-use" href="Testing.md#Specifying-the-format-build-tool_to-use"><span></span></a>Specifying the format (build tool) to use</h1><p>By default, the <code class="code">gyptest.py</code> script will generate configurations for the &ldquo;primary&rdquo; supported build tool for the platform you&#39;re on: Visual Studio on Windows, Xcode on Mac, and (currently) SCons on Linux. An alternate format (build tool) may be specified using the <code class="code">-f</code> option:</p><pre class="code">  $ python gyptest.py -f make test/hello/gyptest-all.py
  PYTHONPATH=/home/knight/src/gyp/trunk/test/lib
  TESTGYP_FORMAT=make
  /usr/bin/python test/hello/gyptest-all.py
  PASSED
  $
</pre><p>Multiple tools may be specified in a single pass as a comma-separated list:</p><pre class="code">  $ python gyptest.py -f make,scons test/hello/gyptest-all.py
  PYTHONPATH=/home/knight/src/gyp/trunk/test/lib
  TESTGYP_FORMAT=make
  /usr/bin/python test/hello/gyptest-all.py
  PASSED
  TESTGYP_FORMAT=scons
  /usr/bin/python test/hello/gyptest-all.py
  PASSED
  $
</pre><h2><a class="h" name="Test-script-functions-and-methods" href="Testing.md#Test-script-functions-and-methods"><span></span></a>Test script functions and methods</h2><p>The <code class="code">TestGyp</code> class contains a lot of functionality intended to make it easy to write tests. This section describes the most useful pieces for GYP testing.</p><p>(The <code class="code">TestGyp</code> class is actually a subclass of more generic <code class="code">TestCommon</code> and <code class="code">TestCmd</code> base classes that contain even more functionality than is described here.)</p><h3><a class="h" name="Initialization" href="Testing.md#Initialization"><span></span></a>Initialization</h3><p>The standard initialization formula is:</p><pre class="code">  import TestGyp
  test = TestGyp.TestGyp()
</pre><p>This copies the contents of the directory tree in which the test script lives to a temporary directory for execution, and arranges for the temporary directory&#39;s removal on exit.</p><p>By default, any comparisons of output or file contents must be exact matches for the test to pass. If you need to use regular expressions for matches, a useful alternative initialization is:</p><pre class="code">  import TestGyp
  test = TestGyp.TestGyp(match = TestGyp.match_re,
                         diff = TestGyp.diff_re)`
</pre><h3><a class="h" name="Running-GYP" href="Testing.md#Running-GYP"><span></span></a>Running GYP</h3><p>The canonical invocation is to simply specify the <code class="code">.gyp</code> file to be executed:</p><pre class="code">  test.run_gyp(&#39;file.gyp&#39;)
</pre><p>Additional GYP arguments may be specified:</p><pre class="code">  test.run_gyp(&#39;file.gyp&#39;, arguments=[&#39;arg1&#39;, &#39;arg2&#39;, ...])
</pre><p>To execute GYP from a subdirectory (where, presumably, the specified file lives):</p><pre class="code">  test.run_gyp(&#39;file.gyp&#39;, chdir=&#39;subdir&#39;)
</pre><h3><a class="h" name="Running-the-build-tool" href="Testing.md#Running-the-build-tool"><span></span></a>Running the build tool</h3><p>Running the build tool requires passing in a <code class="code">.gyp</code> file, which may be used to calculate the name of a specific build configuration file (such as a MSVS solution file corresponding to the <code class="code">.gyp</code> file).</p><p>There are several different <code class="code">.build_*()</code> methods for invoking different types of builds.</p><p>To invoke a build tool with an explicit <code class="code">all</code> target (or equivalent):</p><pre class="code">  test.build_all(&#39;file.gyp&#39;)
</pre><p>To invoke a build tool with its default behavior (for example, executing <code class="code">make</code> with no targets specified):</p><pre class="code">  test.build_default(&#39;file.gyp&#39;)
</pre><p>To invoke a build tool with an explicit specified target:</p><pre class="code">  test.build_target(&#39;file.gyp&#39;, &#39;target&#39;)
</pre><h3><a class="h" name="Running-executables" href="Testing.md#Running-executables"><span></span></a>Running executables</h3><p>The most useful method executes a program built by the GYP-generated configuration:</p><pre class="code">  test.run_built_executable(&#39;program&#39;)
</pre><p>The <code class="code">.run_built_executable()</code> method will account for the actual built target output location for the build tool being tested, as well as tack on any necessary executable file suffix for the platform (for example <code class="code">.exe</code> on Windows).</p><p><code class="code">stdout=</code> and <code class="code">stderr=</code> keyword arguments specify expected standard output and error output, respectively.  Failure to match these (if specified) will cause the test to fail.  An explicit <code class="code">None</code> value will suppress that verification:</p><pre class="code">  test.run_built_executable(&#39;program&#39;,
                            stdout=&quot;expect this output\n&quot;,
							stderr=None)
</pre><p>Note that the default values are <code class="code">stdout=None</code> and <code class="code">stderr=&#39;&#39;</code> (that is, no check for standard output, and error output must be empty).</p><p>Arbitrary executables (not necessarily those built by GYP) can be executed with the lower-level <code class="code">.run()</code> method:</p><pre class="code">  test.run(&#39;program&#39;)
</pre><p>The program must be in the local directory (that is, the temporary directory for test execution) or be an absolute path name.</p><h3><a class="h" name="Fetching-command-output" href="Testing.md#Fetching-command-output"><span></span></a>Fetching command output</h3><pre class="code">  test.stdout()
</pre><p>Returns the standard output from the most recent executed command (including <code class="code">.run_gyp()</code>, <code class="code">.build_*()</code>, or <code class="code">.run*()</code> methods).</p><pre class="code">  test.stderr()
</pre><p>Returns the error output from the most recent executed command (including <code class="code">.run_gyp()</code>, <code class="code">.build_*()</code>, or <code class="code">.run*()</code> methods).</p><h3><a class="h" name="Verifying-existence-or-non_existence-of-files-or-directories" href="Testing.md#Verifying-existence-or-non_existence-of-files-or-directories"><span></span></a>Verifying existence or non-existence of files or directories</h3><pre class="code">  test.must_exist(&#39;file_or_dir&#39;)
</pre><p>Verifies that the specified file or directory exists, and fails the test if it doesn&#39;t.</p><pre class="code">  test.must_not_exist(&#39;file_or_dir&#39;)
</pre><p>Verifies that the specified file or directory does not exist, and fails the test if it does.</p><h3><a class="h" name="Verifying-file-contents" href="Testing.md#Verifying-file-contents"><span></span></a>Verifying file contents</h3><pre class="code">  test.must_match(&#39;file&#39;, &#39;expected content\n&#39;)
</pre><p>Verifies that the content of the specified file match the expected string, and fails the test if it does not.  By default, the match must be exact, but line-by-line regular expressions may be used if the <code class="code">TestGyp</code> object was initialized with <code class="code">TestGyp.match_re</code>.</p><pre class="code">  test.must_not_match(&#39;file&#39;, &#39;expected content\n&#39;)
</pre><p>Verifies that the content of the specified file does <em>not</em> match the expected string, and fails the test if it does.  By default, the match must be exact, but line-by-line regular expressions may be used if the <code class="code">TestGyp</code> object was initialized with <code class="code">TestGyp.match_re</code>.</p><pre class="code">  test.must_contain(&#39;file&#39;, &#39;substring&#39;)
</pre><p>Verifies that the specified file contains the specified substring, and fails the test if it does not.</p><pre class="code">  test.must_not_contain(&#39;file&#39;, &#39;substring&#39;)
</pre><p>Verifies that the specified file does not contain the specified substring, and fails the test if it does.</p><pre class="code">  test.must_contain_all_lines(output, lines)
</pre><p>Verifies that the output string contains all of the &ldquo;lines&rdquo; in the specified list of lines.  In practice, the lines can be any substring and need not be <code class="code">\n</code>-terminaed lines per se.  If any line is missing, the test fails.</p><pre class="code">  test.must_not_contain_any_lines(output, lines)
</pre><p>Verifies that the output string does <em>not</em> contain any of the &ldquo;lines&rdquo; in the specified list of lines.  In practice, the lines can be any substring and need not be <code class="code">\n</code>-terminaed lines per se.  If any line exists in the output string, the test fails.</p><pre class="code">  test.must_contain_any_line(output, lines)
</pre><p>Verifies that the output string contains at least one of the &ldquo;lines&rdquo; in the specified list of lines.  In practice, the lines can be any substring and need not be <code class="code">\n</code>-terminaed lines per se.  If none of the specified lines is present, the test fails.</p><h3><a class="h" name="Reading-file-contents" href="Testing.md#Reading-file-contents"><span></span></a>Reading file contents</h3><pre class="code">  test.read(&#39;file&#39;)
</pre><p>Returns the contents of the specified file.  Directory elements contained in a list will be joined:</p><pre class="code">  test.read([&#39;subdir&#39;, &#39;file&#39;])
</pre><h3><a class="h" name="Test-success-or-failure" href="Testing.md#Test-success-or-failure"><span></span></a>Test success or failure</h3><pre class="code">  test.fail_test()
</pre><p>Fails the test, reporting <code class="code">FAILED</code> on standard output and exiting with an exit status of <code class="code">1</code>.</p><pre class="code">  test.pass_test()
</pre><p>Passes the test, reporting <code class="code">PASSED</code> on standard output and exiting with an exit status of <code class="code">0</code>.</p><pre class="code">  test.no_result()
</pre><p>Indicates the test had no valid result (i.e., the conditions could not be tested because of an external factor like a full file system).  Reports <code class="code">NO RESULT</code> on standard output and exits with a status of <code class="code">2</code>.</p></div></div></div><footer class="Site-footer"><div class="Footer"><div class="Footer-poweredBy">Powered by <a href="https://gerrit.googlesource.com/gitiles/">Gitiles</a></div><div class="Footer-links"></ul></div></footer></body></html>