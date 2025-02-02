# Best_Maple_leaf剧本开发指南

我们使用Nova框架 脚本选用NovaScript脚本语言

## 基本格式

Nova的脚本格式称为NovaScript，主要由文本和Lua代码块组成。文件后缀名一般为`.txt`（需要被Unity识别为text asset），建议用支持Lua语法高亮的编辑器来编辑。

脚本文件一般放在`Assets/Resources/Scenarios/`文件夹下，文件位置在`NovaController.prefab`中由`GameState.scriptPath`设置。

一部作品的脚本由许多节点（node）组成。脚本可以被拆分为多个文件，一个文件中可以有多个节点。脚本解析的结果与读取文件的顺序无关。

每个节点的开头和结尾处各有一个提前代码块（eager execution block），语法为`@<| ... |>`。它记录着关于节点的信息，比如每一章的标题和选项等等，在游戏开始之前执行。

一个节点中可以有许多“条”对话（dialogue entry）。每条对话可以包括一个延迟代码块（lazy execution block），语法为`<| ... |>`，以及一些文本。延迟代码块记录着这条对话对应的演出代码，在游戏过程中执行。

文本中可以用`：：`或`::`分隔角色名称和台词。（台词前后的引号`“”`不是NovaScript语法的一部分，游戏制作者可以选择其他排版习惯。）

角色名称中还可以用`//`分隔显示名称和内部名称，内部名称可以用于自动语音等功能。

文本中可以使用TextMeshPro提供的rich text XML标记。

两条对话之间由一个空行分隔。一条对话的文本可以有许多行。如果文本中需要空行，可以写一个空的XML标记`<b></b>`。

一条对话的延迟代码块和文本之间不应该有空行。（如果有空行，就会变成两条对话，点一下鼠标执行代码，再点一下鼠标播放文本。）

节点开头的提前代码块和第一条对话之间，以及最后一条对话与结尾的提前代码块之间，也不应该有空行。

建议在一个代码块中按以下顺序写代码：角色、背景、前景、时间轴、音乐、音效、对话框、语音、其他系统设置。

`Tools/Scenarios/lint.py`可以用于检查脚本的一些常见问题。

## Lua接口

Nova提供的Lua文件放在`Assets/Nova/Lua/`文件夹下。修改Lua文件之后，打包游戏之前，需要在Unity Editor的上面的菜单中运行`Lua -> Copy Lua Files to Resources`，详见[游戏打包](https://github.com/Lunatic-Works/Nova/wiki/Build)。

我们目前使用的Lua运行库为[ToLua#](https://github.com/topameng/tolua)，它使用的Lua语言版本为LuaJIT，与Lua 5.1完全兼容，Lua 5.2的特性只有一部分兼容。

把C#接口绑定到Lua的方法详见[Lua接口注册](https://github.com/Lunatic-Works/Nova/wiki/Lua-Register)。

## 调试

在Unity Editor中运行游戏时，修改Lua文件或脚本之后，按R可以重新加载脚本，而不用重新运行游戏。

但是由于一些缓存机制，重新加载的脚本可能会出现错误，这时需要在Unity Editor的上面的菜单中运行`Nova -> Clear Save Data`来清空存档。

如果一开始游戏，Unity Editor就闪退，也可以试试清空存档。

## 脚本结构 script_loader.lua

- 设置节点名称

   

  ```
  label(name, display_name)
  ```

  - `name`是一个字符串，表示程序内部使用的节点名称，设置跳转时用的是这个名称
  - 如果`name`以`l_`开头，这个节点名称就是定义在局部的，只能在同一个文件中使用，其他文件中可以有同名的节点
  - `display_name`是一个字符串，表示显示给玩家看的节点名称
  - `display_name`可省略，如果这个文件中还没有定义过`display_name`，则默认与`name`相同，否则默认为上一个`display_name`

- 设置跳转

   

  ```
  jump_to(dest)
  ```

  - `dest`是一个字符串，表示目标节点的名称

- 设置剧情分支

   

  ```
  branch(branches)
  ```

  - `branches`是一个array，每个元素是一个table，它的key有`dest, text, mode, cond`

  - `dest`是一个字符串，表示目标节点的名称

  - `text`是一个字符串，表示显示在按钮上的文本

  - ```
    mode
    ```

    是一个字符串，可以为

    ```
    'normal'|'jump'|'show'|'enable'
    ```

    - `'normal'`表示普通选项，此时`cond`必须省略

    - ```
      'jump'
      ```

      表示如果

      ```
      cond
      ```

      返回true，就会直接跳转到目标节点，否则不显示这个选项，此时

      ```
      text
      ```

      必须省略

      - 如果省略`cond`，则一定会跳转
      - 如果有多个选项是`'jump'`，则会按顺序测试每个`cond`，按照第一个返回true的来跳转

    - `'show'`表示如果`cond`返回true，这个选项才会显示

    - `'enable'`表示如果`cond`返回true，这个选项才能点击

  - ```
    cond
    ```

    是一个函数，没有参数，返回一个boolean

    - 如果`cond`是字符串，就会把它parse成表达式，然后转换成返回这个表达式的函数
    - `cond`必须返回boolean，我们不会进行类型转换，因为Lua会把任何数字和字符串转换为true

  - 具体的例子可以参考`Assets/Resources/Scenarios/test_branch.txt`

- 设置章节

   

  ```
  is_chapter()
  ```

  - 跳到上一个/下一个章节时会在这个节点停止，但是这个节点不一定显示在章节选择界面

- 设置开始节点

   

  ```
  is_start()
  ```

  - 这个节点会显示在章节选择界面

- 设置一开始就解锁的开始节点

   

  ```
  is_unlocked_start()
  ```

  - 同时会设置`is_chapter()`
  - 在标题界面点击“开始游戏”时，如果只有一个开始节点已经解锁，就会跳过章节选择界面，直接进入游戏

- 设置调试节点

  ```
  is_debug()
  ```

  - 调试节点正常情况下不会出现在章节选择界面中，当按Ctrl键解锁所有章节时才会出现
  - 在模拟测试中也可以设置是否解锁调试节点

- 设置结局

   

  ```
  is_end(name)
  ```

  - `name`是一个字符串，表示程序内部使用的结局名称，目前没什么用

- 每个节点开头的提前代码块必须有`label`，可以有`is_start`等；结尾的提前代码块必须有`jump_to, branch, is_end`之一

- 具体的例子可以参考`Assets/Resources/Scenarios/tut01.txt`

## 图像 graphics.lua

- 显示图像

   

  ```
  show(obj, image_name, coord, color, fade)
  ```

  - ```
    obj
    ```

    可以为

    ```
    GameCharacterController
    ```

    （如

    ```
    ergong
    ```

    ）或

    ```
    SpriteController
    ```

    （如

    ```
    bg
    ```

    ）

    - Controller绑定到Lua的名称在Unity Editor中由`*Controller.luaGlobalName`设置

  - ```
    image_name
    ```

    是一个字符串

    - 如果

      ```
      obj
      ```

      是

      ```
      GameCharacterController
      ```

      ，

      ```
      image_name
      ```

      就是所显示的pose的名称

      - 一个pose包括多个立绘部件，在`pose.lua`中设置

    - 如果

      ```
      obj
      ```

      是

      ```
      SpriteController
      ```

      ，

      ```
      image_name
      ```

      就是所显示的图片的名称，如

      ```
      'room'
      ```

      - 图片名称不需要后缀名，应避免名称相同而后缀名不同的素材
      - 图片文件位置由`SpriteController.imageFolder`设置

  - ```
    coord
    ```

    是图像的坐标，格式为

    ```
    {x, y, scale, z, angle}
    ```

    ，元素均为数值

    - 坐标系：画面中心的`x, y, z`为0，向右`x`增大，向上`y`增大，向里`z`增大
    - `angle`的正值为逆时针，范围为-180..180
    - `scale, z, angle`可省略，默认为这个controller之前的值
    - `scale`和`angle`可以改为三个数值的table，表示`x, y, z`分量
    - 为了方便只有2D的演出，我们没有把`z`和`x, y`放在一起

  - ```
    color
    ```

    是与图像相乘的颜色，是一个table，包括1..4个数值，范围为0..1

    - 1个数值：`r, g, b`均为该值，`a`为1
    - 2个数值：`r, g, b`均为第一个值，`a`为第二个值
    - 3个数值：`r, g, b`为这三个数值，`a`为1
    - 4个数值：`r, g, b, a`为这四个数值

  - `fade`是一个boolean，fade为true时，如果这个controller上有自动淡入淡出的脚本，就会进行淡入淡出

  - `coord, color, fade`可省略，默认`coord, color`为这个controller之前的值，`fade`为true

  - 可以在`animation_presets.lua`中定义一些常用的`coord`和`color`

  - 可以先在Unity Editor中移动各种东西来尝试构图，再把数值写到脚本里

- 隐藏图像 `hide(obj, fade)`

- 设置渲染顺序 `set_render_queue(obj, to)`

- 具体的例子可以参考`Assets/Resources/Scenarios/tut02.txt`

## 声音 audio.lua

- 播放音效

   

  ```
  sound(audio_name, volume, pos, use_3d)
  ```

  - `audio_name`是音效的名称，文件位置由`SoundController.audioFolder`设置

  - ```
    volume
    ```

    范围为0..1

    - 实际的音量为脚本中的音量与设置中的音量的乘积

  - ```
    pos
    ```

    是音效的位置，格式为

    ```
    {x, y, z}
    ```

    - 音效素材一般不需要做出“在左边还是在右边”的立体声效果，而是由Unity实现
    - 但是音效一边播放一遍移动的效果可以做到素材里，比如一辆车从左开到右

  - ```
    use_3d
    ```

    是一个boolean，表示是否启用音源的3D效果，如多普勒效应和双耳相位差

    - 即使不启用3D效果，声音仍然会越近越响

  - `volume, pos, use_3d`可省略，默认`volume`为1，`position`为摄像机的位置，`use_3d`为false

- 播放语音

   

  ```
  say(obj, audio_name, delay, override_auto_voice)
  ```

  - `obj`为`GameCharacterController`
  - `audio_name`是语音的名称，文件位置由`GameCharacterController.voiceFolder`设置
  - `delay`表示进入这条对话后延迟多少时间播放语音，以秒为单位
  - `override_auto_voice`是一个boolean，表示是否跳过这条对话的自动语音
  - `delay, override_auto_voice`可省略，默认`delay`为0，`override_auto_voice`为true

- 播放音乐

   

  ```
  play(obj, audio_name, volume)
  ```

  - `obj`为`AudioController`
  - `audio_name`是音乐的名称，文件位置由`AudioController.audioFolder`设置
  - `volume`可省略，默认为0.5（与`sound`的默认`volume`不同）
  - `AudioController`上面的`AudioSource`可以设为不循环播放
  - 可以在`ViewManager.audiosToPause`中设置切换界面时暂停某些`AudioController`，以保证声音和演出动画的同步

- 停止音乐 `stop(obj)`

- 改变音量 `volume(obj, value)`

- 动画改变音量

   

  ```
  anim:volume(obj, value, duration)
  ```

  - `duration`可省略，默认为1

- 动画淡入音乐

   

  ```
  anim:fade_in(obj, audio_name, volume, duration)
  ```

  - `volume, duration`可省略，默认`volume`为0.5，`duration`为1

- 动画淡出音乐

   

  ```
  anim:fade_out(obj, duration)
  ```

  - `duration`可省略，默认为1

- 具体的例子可以参考`Assets/Resources/Scenarios/tut03.txt`

## 对话框 dialogue_box.lua

- 停止自动和快进模式 `stop_auto_ff()`

- 停止快进模式 `stop_ff()`

- 跳到下一条对话 `force_step()`

- 设置对话框

   

  ```
  set_box(pos_name, style_name, auto_new_page)
  ```

  - `pos_name`是对话框位置预设的名称，`style_name`是对话框风格预设的名称，在`dialogue_box.lua`中设置
  - `auto_new_page`是一个boolean，`auto_new_page`为true时，如果对话框的`mode`是`append`，就会执行`new_page()`
  - `pos_name, style_name, auto_new_page`可省略，`pos_name, style_name`的默认值在`dialogue_box.lua`中设置，`auto_new_page`默认为true
  - 要隐藏对话框，可以把对话框的位置移到画面外
  - 预设的`pos_name`中，`'full'`是全屏对话框，`'hide'`是隐藏对话框

- 清空对话框 `new_page()`

- 延迟文本出现 `text_delay(time)`

- 设置文本动画时长 `text_duration(time)`

- 隐藏对话框一段时间后再出现

   

  ```
  box_hide_show(duration, pos_name, style_name)
  ```

  - `duration, pos_name, style_name`可省略，默认`duration`为1
  - 如果使用自动语音，建议对语音设置相应的延时

- 具体的例子可以参考`Assets/Resources/Scenarios/tut04.txt`

## 动画系统的说明

- 一“组”动画由许多“段”动画（animation entry）组成

  - 每段对话可以接在动画的根或上一段动画后面，形成树状结构，用来定义动画的串行和并行播放

- 演出用到的动画分为对话内动画（per dialogue animation）和持续动画（holding animation）

  - 对话内动画只能在一条对话之内播放，点击鼠标时动画会停止到最终状态
  - 持续动画播放的过程可以跨越多条对话，点击鼠标时动画不会停止，必须用脚本停止
  - 两种动画各有一个`NovaAnimation`作为“根”，名称分别为`anim`和`anim_hold`

- `anim:_and()`可以让下一段动画与上一段动画同时开始

- `anim:stop()`可以停止一组动画

- 持续动画开始时要调用

  ```
  anim_hold_begin()
  ```

  ，结束时要调用

  ```
  anim_hold_end()
  ```

  ，以处理动画系统和存档系统的一些状态

  - TODO：等我们的preload系统能更好地parse Lua的时候，就不用手动写`anim_hold_begin`和`anim_hold_end`了

- 可以把一些常用的动画写成函数放在`animation_presets.lua`里

- 详见[演出系统](https://github.com/Lunatic-Works/Nova/wiki/Choreo)

## 动画 animation_high_level.lua

- 动画等待

   

  ```
  anim:wait(duration)
  ```

  - `duration`以秒为单位

- 动画等待另一组动画播放结束 `anim:wait_all(wrap_anim)`

- 动画执行代码

   

  ```
  anim:action(func, ...)
  ```

  - 如果代码只有一个函数，就把函数名和参数当作`action`的参数，比如`anim:action(show, bg, 'room')`

  - 如果代码有很多行，就把代码放在一个函数里，比如：

    ```
    anim:action(function()
            show(bg, 'room')
            play(bgm, 'room')
        end)
    ```

- 动画无限循环

   

  ```
  anim:loop(func)
  ```

  - func的输入为一段动画，输出为接在这段动画后面的一段动画，比如：

    ```
    anim:loop(function(entry)
            return entry:wait(1
                ):action(show, bg, 'room'
                ):wait(1
                ):action(show, bg, 'corridor')
        end)
    ```

- 移动 `move(obj, coord)`

- 动画移动

   

  ```
  anim:move(obj, coord, duration, easing)
  ```

  - `move`等函数如果不接在`:`后面，就不是动画，没有`duration, easing`参数

  - ```
    easing
    ```

    为

    ```
    {start_slope, target_slope}
    ```

    ，表示开始和结束时的速度

    - 0表示开始时从静止加速/结束时减速到静止，1表示匀速运动，可以小于0或大于1
    - 如果两者相同，可以只写一个数
    - 如果需要更复杂的动画曲线，可以写`{func_name, args...}`，并在`animation_high_level.lua`中的`easing_func_name_map`里绑定`func_name`对应的`EasingFunction`，`EasingFunction`在`AnimationEntry.cs`中定义

  - `duration, easing`可省略，默认`duration`为1，`easing`为`{0, 0}`

- 改变颜色 `tint(obj, color)`

- 动画改变颜色 `anim:tint(obj, color, duration, easing)`

- 改变环境颜色

   

  ```
  env_tint(obj, color)
  ```

  - `tint`一般用于短期改变颜色，`env_tint`一般用于黄昏、夜晚等效果，实际的颜色为`tint`与`env_tint`的乘积

- 动画改变环境颜色 `anim:env_tint(obj, color, duration, easing)`

- 具体的例子可以参考`Assets/Resources/Scenarios/tut05.txt`

- 更低层的接口详见`animation.lua`

## 转场与特效 transition.lua

- 第一类转场

   

  ```
  anim:trans(obj, image_name, shader_layer, times, properties, color2)
  ```

  - `obj`可以为`SpriteController`或`CameraController`

  - ```
    image_name
    ```

    - 如果`obj`是`SpriteController`，`image_name`就是转场后图片的名称
    - 如果`obj`是`CameraController`，则进行全局转场（把角色和背景等一起转场），`image_name`是一个函数，定义转场后的内容

  - ```
    shader_layer
    ```

    - 如果

      ```
      obj
      ```

      是

      ```
      SpriteController
      ```

      ，则

      ```
      shader_layer
      ```

      就是

      ```
      shader_name
      ```

      ，是一个字符串，表示shader绑定到Lua的名称

      - 如果shader在Unity中的名称是`Foo Bar`，绑定到Lua之后会自动转换成`foo_bar`
      - `shader_alias_map`可以自定义绑定的名称

    - 如果

      ```
      obj
      ```

      是

      ```
      CameraController
      ```

      ，则

      ```
      shader_layer
      ```

      可以是

      ```
      shader_name
      ```

      或

      ```
      {shader_name, layer_id}
      ```

      - 摄像机上可以叠加多层shader，按`layer_id`从小到大依次渲染
      - 转场默认的`layer_id`为1，特效默认的`layer_id`为0
      - 目前不支持同一种特效叠加多次

  - `times`可以是`duration`或`{duration, easing}`

  - ```
    properties
    ```

    为shader的属性

    - TODO：由于一些历史遗留问题，shader的名称会从Unity风格转换为Lua风格，但是property的名称不会，这里以后要统一

  - ```
    color2
    ```

    为转场后图片的颜色

    - 如果`obj`是`CameraController`，则`color2`无效

  - `times, properties, color2`可省略，默认`times`为`{1, {1, 1}}`，`properties`为`default_shader_properties_map`提供的默认值，`color2`为之前的颜色

  - 改变背景一般不会直接用`show`，总是用转场；但是改变立绘时默认有一个渐变，一般不需要转场

- 第二类转场

   

  ```
  anim:trans2(obj, image_name, shader_layer, times, properties, times2, properties2, color2)
  ```

  - 两类转场的区别：
    - 第一类转场同时显示转场前后的两张图片；第二类转场一次只显示一张图片，先让前一张图片消失，再让后一张图片出现
    - 第二类转场前半段和后半段的`times`和`properties`可以分别设置

- 特效

   

  ```
  vfx(obj, shader_layer, t, properties)
  ```

  - `shader_name`为`nil`或空字符串表示取消特效
  - `t`对应shader的属性`_T`，范围为0..1，0表示特效不出现，1表示特效完全出现
  - `t, properties`可省略，默认`t`为1

- 特效动画

   

  ```
  anim:vfx(obj, shader_layer, start_target_t, times, properties)
  ```

  - ```
    start_target_t
    ```

    为

    ```
    {start_t, target_t}
    ```

    - 如果`target_t`为0，动画结束后会自动取消特效

  - `times, properties`可省略，默认`times`为`{1, {1, 1}}`

- 具体的例子可以参考`Assets/Resources/Scenarios/tut06.txt`

## 时间轴 timeline.lua

- 显示时间轴prefab

   

  ```
  timeline(timelime_name, time)
  ```

  - `time`为时间轴所处的时刻，以秒为单位

  - `time`可省略，默认为0

  - 如果prefab中没有`playableDirector`，则`time`必须省略

  - 如果prefab中有

    ```
    CameraController
    ```

    ，main camera会切换到那个摄像机

    - 在时间轴没有移动那个摄像机时，仍然可以用脚本移动那个摄像机

- 隐藏时间轴prefab `timeline_hide()`

- 跳转时间轴 `timeline_seek(time)`

- 动画播放时间轴 `anim:timeline_play(to, duration, slope)`

## 视频 video.lua

- 显示视频 `video(video_name)`

- 隐藏视频 `video_hide()`

- 动画播放视频

   

  ```
  anim:video_play(duration)
  ```

  - 开始播放视频，然后进行长为`duration`的延时
  - `duration`可省略，默认为视频的长度
  - 从显示视频到开始播放视频之间可能会有卡顿（TODO：我们要考虑一下怎么预加载）

- 具体的例子可以参考`Assets/Resources/Scenarios/test_video.txt`

## 变量 variables.lua

- 在Lua脚本中，`v_`开头的全局变量为Nova的变量，会保存在存档里
- `gv_`开头的全局变量为Nova的全局变量，会保存在全局存档里，不会在读档或回跳时重置
- 变量的类型可以是boolean、number或string
- 把变量设为`nil`就是删除这个变量
- 在文本中可以用`{{变量名}}`显示number或string类型的变量
- 具体的例子可以参考`Assets/Resources/Scenarios/test_variables.txt`和`test_global_variable.txt`

## 警告与通知 alert.lua

- 显示警告 `alert(text)`
- 显示通知 `notify(text)`
- 具体的例子可以参考`Assets/Resources/Scenarios/tut04.txt`

## 自动语音 auto_voice.lua

- 开启自动语音

   

  ```
  auto_voice_on(name, index)
  ```

  - `name`是文本中的角色名称（如`'王二宫'`），而不是Lua变量名（如`ergong`）
  - 如果文本中设置了角色的显示名称和内部名称（如`ch2.txt`中的`？？？//张浅野`），而`name`设为内部名称（`'张浅野'`），这条对话也会播放自动语音
  - `index`是语音文件名中的编号

- 关闭自动语音 `auto_voice_off(name)`

- 关闭所有角色的自动语音 `auto_voice_off_all()`

- 给这条对话的自动语音设置延时

   

  ```
  set_auto_voice_delay(value)
  ```

  - `value`以秒为单位

- 跳过这条对话的自动语音

   

  ```
  auto_voice_skip()
  ```

  - `say`默认会跳过这条对话的自动语音

- 具体的例子可以参考`Assets/Resources/Scenarios/ch1.txt .. ch4.txt`

## 头像 avatar.lua

TODO

- 具体的例子可以参考`Assets/Resources/Scenarios/test_avatar.txt`

