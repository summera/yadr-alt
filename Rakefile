require 'rake'
require 'fileutils'
require File.join(File.dirname(__FILE__), 'bin', 'yadr', 'plug')

desc "Hook our dotfiles into system-standard positions."
task :install => [:submodule_init, :submodules] do
  puts
  puts "======================================================"
  puts "Welcome to YADR Installation."
  puts "======================================================"
  puts

  install_homebrew if RUBY_PLATFORM.downcase.include?("darwin")
  install_gems
  install_python_packages
  install_node_packages

  # this has all the runcoms from this directory.
  file_operation(Dir.glob('git/*')) if want_to_install?('git configs (color, aliases)')
  file_operation(Dir.glob('ag/*')) if want_to_install?('ag configs (sane ag options for searching)')
  file_operation(Dir.glob('pry/*')) if want_to_install?('pry configs (more colorful)')
  file_operation(Dir.glob('ruby/*')) if want_to_install?('rubygems config (faster/no docs)')
  file_operation(Dir.glob('python/*')) if want_to_install?('pylint etc.')
  file_operation(Dir.glob('javascript/*')) if want_to_install?('jshint etc.')
  file_operation(Dir.glob('ctags/*')) if want_to_install?('ctags config (better js/ruby support)')
  file_operation(Dir.glob('tmux/*')) if want_to_install?('tmux config')
  file_operation(Dir.glob('vimify/*')) if want_to_install?('vimification of command line tools')

  if want_to_install?('Hammerspoon (OSX window managing/scripting)')
    if RUBY_PLATFORM.downcase.include? 'darwin'
      file_operation(Dir.glob('hammerspoon'))

      run %{brew install lua luarocks}
      run %{echo 'rocks_servers = { "http://rocks.moonscript.org" }' > ~/.luarocks/config.lua}
    end
  end

  if want_to_install?('vim configuration (highly recommended)')
    file_operation Dir['vim', 'vimrc', 'vimperator', 'vimperatorrc']
    Rake::Task["install_plug"].execute
  end

  Rake::Task["install_prezto"].execute

  install_fonts

  install_term_theme if RUBY_PLATFORM.downcase.include?("darwin")

  run_bundle_config

  success_msg("installed")
end

task :install_prezto do
  if want_to_install?('zsh enhancements & prezto')
    install_prezto
  end
end

task :update do
  Rake::Task["install"].execute
  #TODO: for now, we do the same as install. But it would be nice
  #not to clobber zsh files
end

task :submodule_init do
  unless ENV["SKIP_SUBMODULES"]
    run %{ git submodule update --init --recursive }
  end
end

desc "Init and update submodules."
task :submodules do
  unless ENV["SKIP_SUBMODULES"]
    puts "======================================================"
    puts "Downloading YADR submodules...please wait"
    puts "======================================================"

    run %{
      cd $HOME/.yadr
      git submodule update --recursive
      git clean -df
    }
    puts
  end
end

desc "Runs Plug installer in a clean vim environment"
task :install_plug do
  puts "======================================================"
  puts "Installing and updating plugs."
  puts "The installer will now proceed to run PlugInstall."
  puts "======================================================"

  puts ""

  plug_path = File.join ENV['HOME'], 'vim', 'autoload'
  unless File.exists? plug_path
    run %{
      mkdir -p ~/.vim/autoload
      curl -fLo ~/.vim/autoload/plug.vim \
          https://raw.githubusercontent.com/junegunn/vim-plug/master/plug.vim
    }
  end

  Plug::update_plug
end

task :default => 'install'


private
def run(cmd)
  puts "[Running] #{cmd}"
  `#{cmd}` unless ENV['DEBUG']
end

def number_of_cores
  if RUBY_PLATFORM.downcase.include?("darwin")
    cores = run %{ sysctl -n hw.ncpu }
  else
    cores = run %{ nproc }
  end
  puts
  cores.to_i
end

def run_bundle_config
  return unless system("which bundle")

  bundler_jobs = number_of_cores - 1
  puts "======================================================"
  puts "Configuring Bundlers for parallel gem installation"
  puts "======================================================"
  run %{ bundle config --global jobs #{bundler_jobs} }
  puts
end

def install_node_packages
  return unless system('which npm')

  puts "======================================================"
  puts "Install Node packages"
  puts "======================================================"
  run %{ npm install -g jshint eslint }
  puts
end

def install_python_packages
  return unless system('which pip')
  return unless system('pip3')

  puts "======================================================"
  puts "Install Python packages"
  puts "======================================================"
  run %{ pip install pylint ipython }
  run %{ pip3 install pylint ipython }
  puts
end

def install_gems
  return unless system('which gem')

  puts "======================================================"
  puts "Installing gems for optimized Ruby experience"
  puts "======================================================"
  run %{ gem install pry pry-theme lunchy rubocop ripper-tags }
  puts
end

def install_homebrew
  run %{which brew}
  unless $?.success?
    puts "======================================================"
    puts "Installing Homebrew, the OSX package manager...If it's"
    puts "already installed, this will do nothing."
    puts "======================================================"
    run %{ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)"}
  end

  puts
  puts
  puts "======================================================"
  puts "Updating Homebrew."
  puts "======================================================"
  run %{brew update}
  puts
  puts
  puts "======================================================"
  puts "Installing Homebrew packages...There may be some warnings."
  puts "======================================================"
  run %{brew install zsh ctags git hub tmux reattach-to-user-namespace the_silver_searcher}
  run %{brew install vim --with-lua --with-luajit}
  puts
  puts
end

def install_fonts
  puts "======================================================"
  puts "Installing patched fonts for Powerline/Lightline."
  puts "======================================================"
  run %{ cp -f $HOME/.yadr/fonts/* $HOME/Library/Fonts } if RUBY_PLATFORM.downcase.include?("darwin")
  run %{ mkdir -p ~/.fonts && cp ~/.yadr/fonts/* "$_" && fc-cache -vf ~/.fonts } if RUBY_PLATFORM.downcase.include?("linux")
  puts
end

def install_term_theme
  if !File.exists?(File.join(ENV['HOME'], '/Library/Preferences/com.googlecode.iterm2.plist'))
    puts "======================================================"
    puts "Please check your color settings in iTerm2 to make sure things are swell."
    puts "Preferences> Profiles> [your profile]> Colors> Load Preset.."
    puts "======================================================"
    return
  end

  # Ask the user which theme he wants to install
  message = "Which theme would you like to apply to your iTerm2 profile?"
  color_scheme = ask message, iTerm_available_themes

  return if color_scheme == 'None'

  color_scheme_file = File.join('iTerm2', "#{color_scheme}.itermcolors")

  # Ask the user on which profile he wants to install the theme
  profiles = iTerm_profile_list
  message = "I've found #{profiles.size} #{profiles.size>1 ? 'profiles': 'profile'} on your iTerm2 configuration, which one would you like to apply the color theme to?"
  profiles << 'All'
  selected = ask message, profiles

  if selected == 'All'
    (profiles.size-1).times { |idx| apply_theme_to_iterm_profile_idx idx, color_scheme_file }
  else
    apply_theme_to_iterm_profile_idx profiles.index(selected), color_scheme_file
  end
end

def iTerm_available_themes
   Dir['iTerm2/*.itermcolors'].map { |value| File.basename(value, '.itermcolors')} << 'None'
end

def iTerm_profile_list
  profiles=Array.new
  begin
    profiles <<  %x{ /usr/libexec/PlistBuddy -c "Print :'New Bookmarks':#{profiles.size}:Name" ~/Library/Preferences/com.googlecode.iterm2.plist 2>/dev/null}
  end while $?.exitstatus==0
  profiles.pop
  profiles
end

def ask(message, values)
  puts message
  while true
    values.each_with_index { |val, idx| puts " #{idx+1}. #{val}" }
    selection = STDIN.gets.chomp
    if (Float(selection)==nil rescue true) || selection.to_i < 0 || selection.to_i > values.size+1
      puts "ERROR: Invalid selection.\n\n"
    else
      break
    end
  end
  selection = selection.to_i-1
  values[selection]
end

def install_prezto
  puts
  puts "Installing Prezto (ZSH Enhancements)..."

  run %{ ln -nfs "$HOME/.yadr/zsh/prezto" "${ZDOTDIR:-$HOME}/.zprezto" }

  # The prezto runcoms are only going to be installed if zprezto has never been installed
  file_operation(Dir.glob('zsh/prezto/runcoms/z*'), :copy)

  puts
  puts "Adding zprezto modules and bug-fixes..."
  run %{ ln -nfs "$HOME/.yadr/zsh/prezto-override/zpreztorc" "${ZDOTDIR:-$HOME}/.zpreztorc" }
  run %{ ln -nfs "$HOME/.yadr/zsh/prezto-override/zprofile" "${ZDOTDIR:-$HOME}/.zprofile" }

  puts
  puts "Creating directories for your customizations"
  run %{ mkdir -p $HOME/.zsh.before }
  run %{ mkdir -p $HOME/.zsh.after }
  run %{ mkdir -p $HOME/.zsh.prompts }

  if ENV["SHELL"].include? 'zsh' then
    puts "Zsh is already configured as your shell of choice. Restart your session to load the new settings"
  else
    puts "Setting zsh as your default shell"

    if RUBY_PLATFORM.downcase.include?("darwin")
      if File.readlines("/private/etc/shells").grep("/usr/local/bin/zsh").empty?
        puts "Adding zsh to standard shell list"
        run %{ echo "/usr/local/bin/zsh" | sudo tee -a /private/etc/shells }
      end
    end

    run %{ chsh -s #{`which zsh`} }
  end
end

def want_to_install? (section)
  if ENV["ASK"]=="true"
    puts "Would you like to install configuration files for: #{section}? [y]es, [n]o"
    STDIN.gets.chomp == 'y'
  else
    true
  end
end

def file_operation(files, method = :symlink)
  files.each do |f|
    file = f.split('/').last
    source = "#{ENV["PWD"]}/#{f}"
    target = "#{ENV["HOME"]}/.#{file}"

    puts "======================#{file}=============================="
    puts "Source: #{source}"
    puts "Target: #{target}"

    if File.exists?(target) && (!File.symlink?(target) || (File.symlink?(target) && File.readlink(target) != source))
      puts "[Overwriting] #{target}...leaving original at #{target}.backup..."
      run %{ mv "$HOME/.#{file}" "$HOME/.#{file}.backup" }
    end

    if method == :symlink
      run %{ ln -nfs "#{source}" "#{target}" }
    else
      run %{ cp -f "#{source}" "#{target}" }
    end

    # Temporary solution until we find a way to allow customization
    # This modifies zshrc to load all of yadr's zsh extensions.
    # Eventually yadr's zsh extensions should be ported to prezto modules.
    if file == 'zshrc'
      File.open(target, 'a') do |zshrc|
        zshrc.puts('for config_file ($HOME/.yadr/zsh/*.zsh) source $config_file')
      end
    end

    puts "=========================================================="
    puts
  end
end

def list_vim_submodules
  result=`git submodule -q foreach 'echo $name"||"\`git remote -v | awk "END{print \\\\\$2}"\`'`.select{ |line| line =~ /^vim.bundle/ }.map{ |line| line.split('||') }
  Hash[*result.flatten]
end

def apply_theme_to_iterm_profile_idx(index, color_scheme_path)
  values = Array.new
  16.times { |i| values << "Ansi #{i} Color" }
  values << ['Background Color', 'Bold Color', 'Cursor Color', 'Cursor Text Color', 'Foreground Color', 'Selected Text Color', 'Selection Color']
  values.flatten.each { |entry| run %{ /usr/libexec/PlistBuddy -c "Delete :'New Bookmarks':#{index}:'#{entry}'" ~/Library/Preferences/com.googlecode.iterm2.plist } }

  run %{ /usr/libexec/PlistBuddy -c "Merge '#{color_scheme_path}' :'New Bookmarks':#{index}" ~/Library/Preferences/com.googlecode.iterm2.plist }
  run %{ defaults read com.googlecode.iterm2 }
end

def success_msg(action)
  puts ""
  puts "   _     _           _         "
  puts "  | |   | |         | |        "
  puts "  | |___| |_____  __| | ____   "
  puts "  |_____  (____ |/ _  |/ ___)  "
  puts "   _____| / ___ ( (_| | |      "
  puts "  (_______\_____|\____|_|      "
  puts ""
  puts "YADR has been #{action}. Please restart your terminal and vim."
end
