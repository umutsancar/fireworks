# TODO: Build MXP for CS3 and CS4, even if it has to be installed with CS4

require "rubygems"
require "rake"
require "rdiscount"
require "erb"
require 'fileutils'

ORANGE_COMMANDS_VERSION = "0.9.9"

HOME = ENV["HOME"]
CS3 = HOME + "/Library/Application Support/Adobe/Fireworks CS3/"
CS4 = HOME + "/Library/Application Support/Adobe/Fireworks CS4/"
destinations = [CS3,CS4]

COMMANDS_TEMPLATE = <<-EOF
<dynamic_commands><% @commands.each do |command| %>
\t\t<jscommand name="<%= command.name %>" count="1" >
\t\t\t<shortcut text="<%= command.modifier %> <%= command.key %>" />
\t\t</jscommand><% end %>
\t</dynamic_commands>
EOF

MXI = <<-XML
<?xml version="1.0" encoding="UTF-8"?>
<macromedia-extension name="Orange Commands <%= @version %>" version="<%= ORANGE_COMMANDS_VERSION %>" type="command" requires-restart="true">
  <author name="Ale Muñoz" />
  <products>
    <product name="Fireworks" version="8" primary="true" />
  </products>
  <description>
    <![CDATA[
    An amazingly wonderful collection of Commands for Fireworks :)
    ]]>
  </description>
  <ui-access>
    <![CDATA[
<%= @documentation %>
    ]]>
  </ui-access>
  <files>
<% @files.each do |filename| %>    <file source="<%= filename %>" destination="$fireworks/Configuration/<%= filename %>" />
<% end %>
  </files>
</macromedia-extension>
XML

desc "Build MXI file with Commands"
task :mxi => [:clean] do
  # CS3 Extension Manager does not use HTML
  # @documentation = File.read("README.markdown")
  @documentation = RDiscount.new(File.read("README.markdown")).to_html.gsub(/<h(\d+)>/,"<b>").gsub(/<\/h(\d+)>/,"</b>")
  # @documentation = RDiscount.new(File.read("README.markdown")).to_html.gsub(/^\n/,"").gsub(/<h(\d+)>/,"<b>").gsub(/<\/h(\d+)>/,"</b>")
  @files = Dir["Commands/**/**.jsf","Commands/**/**.js"].reject { |o| (o =~ /Development/) }
  open("OrangeCommands_#{ORANGE_COMMANDS_VERSION}.mxi","w") do |f|
    f << ERB.new(MXI).result
  end
end

desc "Build XML for keyboard shortcuts"
task :shortcuts do
  running_folder = %x(pwd).chomp
  xml_source_dirs = [
    "/Applications/Adobe Fireworks CS3/Adobe Fireworks CS3.app/Contents/Resources/en.lproj/Keyboard Shortcuts/",
    "/Applications/Adobe Fireworks CS4/Adobe Fireworks CS4.app/Contents/Resources/en.lproj/Keyboard Shortcuts/"
  ]
  xml_target_dirs = [
    "en/Keyboard\ Shortcuts/CS3",
    "en/Keyboard\ Shortcuts/CS4"
  ]

  LINE_REGEXP = /<dynamic_commands \/>|<dynamic_commands >(.+)<\/dynamic_commands>/ # cr(ap|ee)py

  MODIFIERS = {
    :CTRL => 8,
    :COMMAND => 4,
    :SHIFT => 2,
    :ALT => 1
  }
  KEYCODES = {
    :kLEFT => "28",
    :kRIGHT => "29",
    :kUP => "30",
    :kDOWN => "31",

    :k0 => "48", 
    :k1 => "49",
    :k2 => "50",
    :k3 => "51",
    :k4 => "52",
    :k5 => "53",
    :k6 => "54",
    :k7 => "55",
    :k8 => "56",
    :k9 => "57",

    :kA => "65",
    :kB => "66",
    :kC => "67",
    :kD => "68",
    :kE => "69",
    :kF => "70",
    :kG => "71",
    :kH => "72",
    :kI => "73",
    :kJ => "74",
    :kK => "75",
    :kL => "76",
    :kM => "77",
    :kN => "78",
    :kO => "79",
    :kP => "80",
    :kQ => "81",
    :kR => "82",
    :kS => "83",
    :kT => "84",
    :kU => "85",
    :kV => "86",
    :kW => "87",
    :kX => "88",
    :kY => "89",
    :kZ => "90",
  }

  class Command
    attr_accessor :name
    attr_accessor :modifier
    attr_accessor :key

    def initialize(command_path,shortcut_line)
      self.name = File.basename(command_path,".jsf")
      modifier = 0
      parts = shortcut_line.match(/shortcut: (.*)/)[1].split(" + ")
      self.key = KEYCODES["k#{parts.pop}".to_sym]
      parts.each do |item|
        if MODIFIERS[item.to_sym]
          modifier += MODIFIERS[item.to_sym]
        end
      end
      self.modifier = modifier
    end
  end

  @commands = []

  Dir["Commands/**/**.jsf"].each do |f|
    File.readlines(f).each do |line|
      if line =~ /shortcut:/
        @commands << Command.new(f,line)
        break
      end
    end
  end
  new_commands = ERB.new(COMMANDS_TEMPLATE).result(binding)

  xml_target_dirs.each do |dir|
    if !File.exist? dir
      FileUtils.mkdir_p dir
    end
  end

  xml_source_dirs.each_with_index do |folder,i|
    Dir["#{folder}/*.xml"].each do |f|
      cp f, "#{xml_target_dirs[i]}/"
      file_name = File.basename(f,".xml")
      puts "Generating #{file_name} + Extras"
      file_contents = File.read(f)
      open("#{xml_target_dirs[i]}/#{file_name}.xml","w") do |new_file|
        new_file << file_contents.gsub(LINE_REGEXP,new_commands)
      end
      mv "#{xml_target_dirs[i]}/#{file_name}.xml", "#{xml_target_dirs[i]}/#{file_name} + Extras.xml"
    end
  end
end

desc "Build MXP files"
task :mxp do
  %x(open -a "/Applications/Adobe Extension Manager/Extension Manager.app" OrangeCommands_#{ORANGE_COMMANDS_VERSION}.mxi)
  %x("/Applications/Adobe Extension Manager CS4/Adobe Extension Manager CS4.app/Contents/MacOS/Adobe Extension Manager CS4" -package mxi="OrangeCommands_#{ORANGE_COMMANDS_VERSION}.mxi" mxp="OrangeCommands_CS4.mxp")
end

task :clean do
  FileUtils.rm Dir.glob("*.mxi")
  FileUtils.rm Dir.glob("*.mxp")
end

task :default => [:clean, :shortcuts,:mxi,:mxp]