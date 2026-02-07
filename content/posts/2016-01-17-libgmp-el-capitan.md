---
title:  "libgmp problem on El Capitan"
slug: libgmp-el-capitan
date:   2016-01-17 18:39:16
draft: false
categories: ["osx", "ruby"]
---
After upgrading to El Capitan I ran into a weird issue trying to update the `github-pages` Ruby gem. It failed saying it could not find [libgmp](https://gmplib.org), but I knew it was installed and updating it to the latest version didn't help either.

Here's the output from bundler:

    skalera:pmenglund.github.io martin$ bundle
    Fetching gem metadata from https://rubygems.org/..........
    Fetching version metadata from https://rubygems.org/...
    Fetching dependency metadata from https://rubygems.org/..
    Installing RedCloth 4.2.9 with native extensions

    Gem::Ext::BuildError: ERROR: Failed to build gem native extension.

        /usr/local/var/rbenv/versions/2.2.3/bin/ruby -r ./siteconf20160117-27109-1clpfph.rb extconf.rb
    checking for main() in -lc... *** extconf.rb failed ***
    Could not create Makefile due to some reason, probably lack of necessary
    libraries and/or headers.  Check the mkmf.log file for more details.  You may
    need configuration options.

    Provided configuration options:
    	--with-opt-dir
    	--without-opt-dir
    	--with-opt-include
    	--without-opt-include=${opt-dir}/include
    	--with-opt-lib
    	--without-opt-lib=${opt-dir}/lib
    	--with-make-prog
    	--without-make-prog
    	--srcdir=.
    	--curdir
    	--ruby=/usr/local/var/rbenv/versions/2.2.3/bin/$(RUBY_BASE_NAME)
    	--with-redcloth_scan-dir
    	--without-redcloth_scan-dir
    	--with-redcloth_scan-include
    	--without-redcloth_scan-include=${redcloth_scan-dir}/include
    	--with-redcloth_scan-lib
    	--without-redcloth_scan-lib=${redcloth_scan-dir}/lib
    	--with-clib
    	--without-clib
    /usr/local/var/rbenv/versions/2.2.3/lib/ruby/2.2.0/mkmf.rb:456:in `try_do': The compiler failed to generate an executable file. (RuntimeError)
    You have to install development tools first.
    	from /usr/local/var/rbenv/versions/2.2.3/lib/ruby/2.2.0/mkmf.rb:541:in `try_link0'
    	from /usr/local/var/rbenv/versions/2.2.3/lib/ruby/2.2.0/mkmf.rb:556:in `try_link'
    	from /usr/local/var/rbenv/versions/2.2.3/lib/ruby/2.2.0/mkmf.rb:735:in `try_func'
    	from /usr/local/var/rbenv/versions/2.2.3/lib/ruby/2.2.0/mkmf.rb:966:in `block in have_library'
    	from /usr/local/var/rbenv/versions/2.2.3/lib/ruby/2.2.0/mkmf.rb:911:in `block in checking_for'
    	from /usr/local/var/rbenv/versions/2.2.3/lib/ruby/2.2.0/mkmf.rb:351:in `block (2 levels) in postpone'
    	from /usr/local/var/rbenv/versions/2.2.3/lib/ruby/2.2.0/mkmf.rb:321:in `open'
    	from /usr/local/var/rbenv/versions/2.2.3/lib/ruby/2.2.0/mkmf.rb:351:in `block in postpone'
    	from /usr/local/var/rbenv/versions/2.2.3/lib/ruby/2.2.0/mkmf.rb:321:in `open'
    	from /usr/local/var/rbenv/versions/2.2.3/lib/ruby/2.2.0/mkmf.rb:347:in `postpone'
    	from /usr/local/var/rbenv/versions/2.2.3/lib/ruby/2.2.0/mkmf.rb:910:in `checking_for'
    	from /usr/local/var/rbenv/versions/2.2.3/lib/ruby/2.2.0/mkmf.rb:961:in `have_library'
    	from extconf.rb:5:in `<main>'

    extconf failed, exit code 1

    Gem files will remain installed in /usr/local/var/rbenv/versions/2.2.3/lib/ruby/gems/2.2.0/gems/RedCloth-4.2.9 for inspection.
    Results logged to /usr/local/var/rbenv/versions/2.2.3/lib/ruby/gems/2.2.0/extensions/x86_64-darwin-14/2.2.0-static/RedCloth-4.2.9/gem_make.out

Looking in the `mkmf.log` I saw:

    ld: library not found for -lgmp

which didn't make sense, but then I remembered that I had seen a similar issue when I upgraded to Yosemite, so I ran this to install XCode commandline tools:

    xcode-select --install

which solved the problem!
