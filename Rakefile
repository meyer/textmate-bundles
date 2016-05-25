require 'rake'
require 'shellwords'

PWD = File.dirname(__FILE__)
DeltaDir = File.join(PWD, "delta")

AvianDir = File.expand_path('~/Library/Application Support/Avian')
BundleDir = File.join(AvianDir, 'Bundles')
TMSupportDir = File.expand_path('~/Library/Application Support/TextMate')

def symlink_maybe(src, dest)
  begin
    if File.exist?(dest)
      if File.symlink?(dest)
        rm dest
      elsif File.directory?(dest)
        raise "#{dest} is a directory?!"
      else
        raise "Something already exists at #{dest}"
      end
    end

    puts "Symlinking!"
    ln_s src, dest
  rescue Errno::EEXIST
    puts "File already exists at #{dest}"
  rescue
    puts "Something went wrong"
  end
end

task :repo2submodule do
  `git reset`
  Dir.glob("{src,delta,third-party}/*-tmbundle").each do |repo|
    if File.exist?(File.join(repo, '.git'))
      Dir.chdir(repo)
      remote_url = `git config --get remote.origin.url`.chomp
      Dir.chdir PWD
      system "git submodule add #{remote_url.shellescape} #{repo.shellescape}"
    end
  end
end

desc "Migrate delta bundles to this repository"
task :migrate_deltas do
  migrated_count = 0
  Dir.glob("#{BundleDir}/*.tmbundle").each do |bun|
    if File.symlink?(bun)
      # puts "#{bun} -- ignoring symlink"
      next
    end

    plist_filename = File.join(bun, "info.plist")
    begin
      plist = IO.read(plist_filename)
      next unless !!(/isDelta<\/key>\s+<true/.match(plist))
    rescue
      puts "Error reading '#{plist_filename}'"
    end

    bun_base = File.basename(bun, ".tmbundle")
    bun_dest = File.join(DeltaDir, "#{bun_base}-tmbundle")

    if File.exist?(bun_dest)
      puts "File already exists at '#{bun_dest}'"
      next
    end

    mv(bun, bun_dest)
    symlink_maybe(bun_dest, bun)

    puts bun_dest
  end
end

desc 'Update TextMate bundle symlinks'
task :update_symlinks do
  # Symlink keybindings
  mkdir_p TMSupportDir
  kbSrc = File.expand_path('./src/KeyBindings.dict')
  kbDest = File.join(TMSupportDir, 'KeyBindings.dict')
  symlink_maybe(kbSrc, kbDest)

  # Symlink bundles
  mkdir_p BundleDir
  Dir.chdir(PWD)
  Dir.glob("{src,delta,third-party}/*-tmbundle").each do |d|
    src_path = File.expand_path(d)
    bundle_name = File.basename(d).gsub('-tmbundle', '.tmbundle')
    dest_path = File.join(BundleDir, bundle_name)
    symlink_maybe(src_path, dest_path)
  end
end

task :rename_bundles do
  print "Quit TextMate, then hit enter: "
  STDIN.gets

  Dir.chdir(BundleDir)
  Dir.glob("*.tmbundle").each do |b|
    mv b, "#{File.basename(b, ".tmbundle")}.tmp.tmbundle"
  end

  print "Restart TextMate, quit it again (I know, I know), then hit enter: "
  STDIN.gets

  Dir.glob("*.tmp.tmbundle").each do |b|
    mv b, "#{File.basename(b, ".tmp.tmbundle")}.tmbundle"
  end

  puts "Restart TextMate. Your bundles should be back again."
end

task :ls do
  system "ls -la #{BundleDir.shellescape}"
end

task :open do
  `open #{BundleDir.shellescape}`
  # `open #{TMSupportDir.shellescape}`
end

task :default => [:update_symlinks]
