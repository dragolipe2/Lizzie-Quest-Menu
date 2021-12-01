#===============================================================================
#  *  Quest Menu
#  v. 1.00
#  Feb. 2nd, 2013
#  Written for: www.gdunlimited.net
#  Credits:  Lizzie S
#  Thanks to:  Moonpearl, for a small pearl of wisdom.
#              Marked, for keeping GDUnimited alive
#              RGangsta, for ACE.
#-------------------------------------------------------------------------------
#  Introduction:  This script allows a good quest log for your game, allowing
#                 the player to track quests, see how many they've completed,
#                 etc.
#-------------------------------------------------------------------------------
#  Features:
#    Keeps a tally of all known quests and how many have been completed.
#    Allows you to add or remove objectives.
#-------------------------------------------------------------------------------
#  Instructions:
#   • Place the script in the 'Materials' section.
#   • The script adds the first objective of the first mission automatically.
#   • You must input all the information of your missions in the module marked:
#       Missions
#       There is an example in place.
#   • To call the script, use: SceneManager.call(Scene_Mission)
#   • Use the following syntaxes to affect the script:
#       To add a mission:               $game_party.add_mis(mission_id)
#       To add an objective:            $game_party.add_obj(mission_id, objective_id)
#       To remove an objective:         $game_party.rem_obj(mission_id, objective_id)
#       To mark an objective complete:  $game_party.add_comp_obj(mission_id, objective_id)
#       To mark a mission as complete:  $game_party.comp_mis(mission_id)
#===============================================================================
 
 
#===============================================================================
#  *  Module
#===============================================================================
module Missions
 
  #-----------------------------------------------------------------------------
  #  Mission_Names = { mission_id => "Mission_Name"
  #-----------------------------------------------------------------------------
  Mission_Names = { 0 => "Fetch a Pail of Water",
                    1 => "tch a Pail of Water"
                  }
  #-----------------------------------------------------------------------------
  #  Mission_Desc = { mission_id => ["Description", ["sprite_sheet", index, "name"]]
  #-----------------------------------------------------------------------------
  Mission_Desc = { 0 => ["You have been asked by your mother to fetch a pail of water.",
                                ["Actor2", 1, "Mother"]],
                    1 => ["You have been asked by your mother to fetch a pail of water.",
                                ["Actor2", 1, "Mother"]]
                        }
  #-----------------------------------------------------------------------------
  # Mission_Objectives = {mission_id => {obj_id => "Objective_text",...  }
  #-----------------------------------------------------------------------------
  Mission_Objectives = { 0 => {0 => "You need to go out and find a bucket, wooden is preferable..",
                               1 => "Find a nice sturdy rope. Not something that will decay easily in water. Just something more like nylon.",
                               2 => "Fetch water from the well."
                               },
                         1 => {0 => "a need to go out and find a bucket, wooden is preferable..",
                               1 => "a a nice sturdy rope. Not something that will decay easily in water. Just something more like nylon.",
                               2 => "Fetch water from the well."
                               }
                       }
     
end
#===============================================================================
# END Module
#===============================================================================
 
 
#===============================================================================
#  *  Game_Party
#===============================================================================
class Game_Party < Game_Unit
 
  attr_accessor :missions
  attr_accessor :comp_missions
  attr_accessor :act_obj
  attr_accessor :comp_obj
 
  alias lizzie_missions_gameparty_init initialize
 
  def initialize
    lizzie_missions_gameparty_init
    # Mission_IDs
    @missions = []
    #  Mission_Ids
    @comp_missions = []
    # Mission_Name => [Current_Objectives]
    @act_obj = {}
    @comp_obj = {}
  end
 
  def add_mis(id)
    if @missions.include?(id)
      return
    end
    @missions.push(id)
    mis = Missions
    @act_obj[id] = [0]
    @comp_obj[id] = []
    return
  end
 
  def rem_obj(id, obj_n)
    if @act_obj[id] != nil
      if @act_obj[id].include?(obj_n)
        @act_obj[id].delete(obj_n)
      end
    end
    return
  end
 
 
  def add_obj(id, obj_n)
    mis = Missions
    if @act_obj[id].include?(obj_n)
      return
    end
    @act_obj[id].push(obj_n)
    return
  end
 
 
  def add_comp_obj(id, obj_n)
    for i in 0...@act_obj[id].size
      if @act_obj[id][i] == obj_n
        @act_obj[id].delete(obj_n)
        @comp_obj[id].push(obj_n)
      end
    end
    return
  end
 
  def comp_mis(id)
    if @comp_missions.include?(id)
      return
    end
    @comp_missions.push(id)
    return
  end
 
end
#===============================================================================
# END Game_Party
#===============================================================================
 
 
#===============================================================================
#  *  Window_MissionInfo
#===============================================================================
class Window_MissionInfo < Window_Base
  def initialize
    super(0, 0, 416, 48)
    activate
    refresh
  end
 
 
  def refresh
    contents.clear
    contents.draw_text(0, 0, self.width - 24, 24, "Use L and R to scroll description.")
  end
end
#===============================================================================
#  *  Window_MissionInfo
#===============================================================================
 
 
#===============================================================================
#  *  Window_MissionComp
#===============================================================================
class Window_MissionComp < Window_Base
  def initialize
    super(416, 0, 128, 48)
    self.active = false
    refresh
  end
 
  def refresh
    self.contents.clear
    x = $game_party.missions.size.to_s
    y = $game_party.comp_missions.size.to_s
    self.contents.draw_text(0, 0, self.width - 24, 24, y + "/" + x, 2)
  end
end
#===============================================================================
# END Window_MissionComp
#===============================================================================
 
 
#===============================================================================
#  *  Window_Missions
#===============================================================================
class Window_Missions < Window_Selectable
  def initialize
    super(0, 48, 256, 368)
    self.index = 0
    activate
    @data = []
    refresh
  end
 
  def item_max
    $game_party.missions.size
  end
 
  def mission
    return @data[index]
  end
 
  def refresh
    self.contents.clear
    mis = Missions
    for i in 0...$game_party.missions.size
      x = $game_party.missions[i]
      @data.push(x)
      if $game_party.comp_missions.include?(x)
        change_color(Color.new(128, 128, 128, 255))
      else
        change_color(Color.new(255, 255, 255, 255))
      end
      self.contents.draw_text(4, i * 24, self.width - 24, 24, mis::Mission_Names[x])
    end
  end
 
 
end
#===============================================================================
# END Window_Missions
#===============================================================================
 
 
#===============================================================================
#  *  Window_MissionDesc
#===============================================================================
class Window_MissionDesc < Window_Selectable
  def initialize
    super(256, 48, 288, 368)
    self.index = -1
    activate
  end
 
  #-----------------------------------------------------------------------------
 
  def refresh(id)
    self.contents.clear
    if id == nil
      return
    end
    mis = Missions
   
    @lines = []
    @text_location_y = 0
    @comp_lines = []
   
    desc_text = mis::Mission_Desc[id][0]
    lines_update(desc_text)
   
    @lines.push("")
   
    for i in 0...$game_party.act_obj[id].size
      act_obj_text = " • " + mis::Mission_Objectives[id][$game_party.act_obj[id][i]]
      lines_update(act_obj_text)
    end
   
    if $game_party.act_obj[id].size != 0
      @lines.push("")
    end
   
    for i in 0...$game_party.comp_obj[id].size
      comp_obj_text = " • " +  mis::Mission_Objectives[id][$game_party.comp_obj[id][i]]
      comp_lines_update(comp_obj_text)
    end
   
    set_height
    self.contents = Bitmap.new(264, @n)
   
    draw_character(mis::Mission_Desc[id][1][0], mis::Mission_Desc[id][1][1], 24, 32)
    draw_text(48, 0, 240, 24, mis::Mission_Desc[id][1][2])
   
    for i in 0...@lines.size
      draw_text(0, i * 24 + 48, 264, 24, @lines[i])
      @text_location_y += 24
    end
   
    change_color(Color.new(128, 128, 128, 255))
   
    for i in 0...@comp_lines.size
      draw_text(0, i * 24 + 48 + @text_location_y, 264, 24, @comp_lines[i])
    end
  end
 
  #-----------------------------------------------------------------------------
 
  def lines_update(text)
    string = ""
    for word in text.split
      word_width = self.text_size(word).width + self.text_size(string).width + self.text_size(" ").width
      if word_width > (width - 24)
        @lines.push(string)
        string = ""
      end
      string += word + " "
    end
    unless @lines.include?(string)
      @lines.push(string)
      string = ""
    end
  end
 
  #-----------------------------------------------------------------------------
 
  def comp_lines_update(text)
    string = ""
    for word in text.split
      word_width = self.text_size(word).width + self.text_size(string).width + self.text_size(" ").width
      if word_width > (width - 24)
        @comp_lines.push(string)
        string = ""
      end
      string += word + " "
    end
    unless @comp_lines.include?(string)
      @comp_lines.push(string)
      string = ""
    end
  end
 
  #-----------------------------------------------------------------------------
 
  def set_height
    @n = (@lines.size + @comp_lines.size + 2) * 24
  end
 
end
#===============================================================================
# END Window_MissionDesc
#===============================================================================
 
 
#===============================================================================
#  *  Scene_Mission
#===============================================================================
class Scene_Mission < Scene_MenuBase
 
  alias liz_mission_self_update update_basic
 
  def start
    super
    create_list_window
    create_help_window
    create_comp_window
    create_desc_window
  end
 
  def create_help_window
    @help_window = Window_MissionInfo.new
  end
   
  def create_comp_window
    @comp_window = Window_MissionComp.new
  end
   
  def create_desc_window
    @desc_window = Window_MissionDesc.new
    @desc_window.refresh(@list_window.mission)
  end
 
  def create_list_window
    @list_window = Window_Missions.new
    @list_window.set_handler(:cancel, method(:return_scene))
    @list_window.set_handler(:pagedown, method(:scroll_down))
    @list_window.set_handler(:pageup, method(:scroll_up))
  end
 
  #--------------------------------------------------------------------------
  # Retornar Scene_Item
  #--------------------------------------------------------------------------
  def command_return
    SceneManager.call(Scene_Item)
  end
  
  def scroll_down
    @list_window.activate
    @list_window.cursor_pagedown
    @desc_window.refresh(@list_window.mission)
  end
 
  def update_basic
    liz_mission_self_update
    if Input.trigger?(Input::UP) or Input.trigger?(Input::DOWN)
      @desc_window.refresh(@list_window.mission)
    end
  end
 
  def scroll_up
    @list_window.activate
    @list_window.cursor_pageup
    @desc_window.refresh(@list_window.mission)
  end
end
