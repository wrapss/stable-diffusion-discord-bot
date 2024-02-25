# Stable Diffusion Discord Bot
forked from https://github.com/AndBobsYourUncle/stable-diffusion-discord-bot
at 2023-04-08T07:10:00 (JST)

この時点のやつをフォークした。Go言語もDiscordクライアントも初なので分かるところだけ自分用に進める

I'm new on Go programming language and Discord client development.

Update: my version is not compatible with the upstream version, especially database columns.
        (if you have switched from his version, put my version to another folder.) 

## 001. changed button order and icon captions
at iPhone Discord client, result buttons was not lined up

iPhoneの Dicordクライアントで見づらかったのでボタン配置とキャプションを変更した

![button orders before](https://github.com/pitapan5376/stable-diffusion-discord-bot/blob/master/document/001_change_button1.jpg?raw=true)

shorten caption and swap button orders like midjourney/nijijourney

![rearranged button order](https://github.com/pitapan5376/stable-diffusion-discord-bot/blob/master/document/001_change_button2.jpg?raw=true)

## 002. changed fonts
prompt showing in monospace in Discord client

打ち間違いとかするのでプロンプトはMonospaceで表示するようにした

![change prompt font to monospace](https://github.com/pitapan5376/stable-diffusion-discord-bot/blob/master/document/002_change_prompt_font.png?raw=true)


## 003. enable aspect ratio (without upscaler)
--ar param is parsed and computed new values, but actual result was still default 512x512. 
new width, height is fed to Image Generation param.
this is not using upscalers nor hires.fix, --ar 1:2 gives you 512x1024 so requires enough GPU memory.

ソース内にアスペクト比の機能が書いてあったけど512x512のまま動作しなかったのでなんか計算されてた値を渡せるようにした。
素の値なのでGPU必要（hires.fixとかupscaleはされてないっぽい）

### 1girl --ar 4:3
![sample for ar 4:3](https://github.com/pitapan5376/stable-diffusion-discord-bot/blob/master/document/003_aspect_ratio_4_3.png?raw=true)

### 1girl --ar 1:2
![sample for ar 1:2](https://github.com/pitapan5376/stable-diffusion-discord-bot/blob/master/document/003_aspect_ratio_1_2.png?raw=true)


## 004. add sampling steps
It seems difficult to add parameter like 'prompt:' on Discord ways, so I just copy & modified along --ar parameter.
with --step X gives you result in X steps processed. 
default value is 20, if not specified --step parameter. (sampling method is default Euler_a yet)

Discord風に prompt: とかでパラメータ渡すやり方が難しそうだったのでアスペクト比を取っているやつをコピーしてステップ数を渡せるようにした

### --step 7(512x512)
low steps at 7, seems noisy but It works.

![sample for step 7](https://github.com/pitapan5376/stable-diffusion-discord-bot/blob/master/document/004_steps_param_7.png?raw=true)

### --step 50 --ar 2:1(1024x768)
combo with aspect ratio, I could make AI output more detailed result. 

アスペクト比を横長とかにしたとき、ステップ数を増やすことができるようになったので、画像が荒くなくなった。

![sample for ar 1:2](https://github.com/pitapan5376/stable-diffusion-discord-bot/blob/master/document/004_steps_param_50.png?raw=true)

## 005. add CFG scale parameter
passing CFG scale value. 1.0(min.) to 30.0(max.); the limit is along with AUTO1111 WebUI.

--cfgscale で CFGスケール値を渡せるようにした。数値の許容範囲はAUTO1111のWebUIにあわせた。

image is generated by random seed at current, it's little bit hard to check effectiveness of CFG values.

seedの指定がまだできないので簡単には比較できないが、極端な値にしたら絵柄変わったので多分動いている。

### --cfgscale 1.2
![sample for CFG scale low](https://github.com/pitapan5376/stable-diffusion-discord-bot/blob/master/document/005_cfg_scale_1.png?raw=true)

### --cfgscale 15.3
![sample for CFG scale high](https://github.com/pitapan5376/stable-diffusion-discord-bot/blob/master/document/005_cfg_scale_15.png?raw=true)

## 006. seed parameter
passing seed parameter. default was -1(random); max value 12345678901234567890 is accepted by WebUI, but datatype of the code is int.
valid range of this program should be 0 - 2147483647.(Golang int)

seed値を渡せるようにした。StableDiffusionでの最大値を調べたが、はっきりしたことは不明だった。
実際にWebUIで適当に長い数字を入れると 12345678901234567890 までは受け付けて、桁を増やすとPythonのlong型でオーバーフローしているようだった。
Go言語の実装でint型になっているようなのでその値を最大とした。
実用上困らないと考えてがんばらないことにした。

### --seed 111
同じシード値で同じ結果が出ることが確認できた。

![sample for seed](https://github.com/pitapan5376/stable-diffusion-discord-bot/blob/master/document/006_seed.png?raw=true)

### display seed value when upscaled
when no seed specified (at random), indicates clearly saying random

シード値未指定のときに -1 の表示になっていたので作成後メッセージで random(-1)表記にした

![indicate as random seed value](https://github.com/pitapan5376/stable-diffusion-discord-bot/blob/master/document/006_seed2.png?raw=true)

when upscaling, display seed value at post-generate message

アップスケールのときに作成後メッセージに seed を表示するようにした

![show seed value at upscale](https://github.com/pitapan5376/stable-diffusion-discord-bot/blob/master/document/006_seed3.png?raw=true)

## 007. negative_prompt: param

passing negative_prompt as optional parameter.
default value is hard-coded one: 
  "ugly, tiling, poorly drawn hands, poorly drawn feet, poorly drawn face, out of frame,
   mutation, mutated, extra limbs, extra legs, extra arms, disfigured, deformed, cross-eye,
   body out of frame, blurry, bad art, bad anatomy, blurred, text, watermark, grainy"

ネガティブプロンプトを与えられるようにした。

初期値（未指定）はハードコーディングされている上記の値とみなす。何か値を与えるとそれのみをネガティブプロンプトとする

![negative prompt param](https://github.com/pitapan5376/stable-diffusion-discord-bot/blob/master/document/007_negative_prompt.png?raw=true)

最初は下のURLを参考に全部入れようとしたがうまくいかなかったので、まずネガティブプロンプトだけにした。

https://github.com/AndBobsYourUncle/stable-diffusion-discord-bot/pull/18/commits/cf6ec0d52461d0d2eaac2b5fd98316f88c14b43b


## 008. BUGFIX: seed value for big int

very few times SD api returns seed value as large int value which is over int32.

seed未指定で生成していると intの最大値を超えるものが来たときbotが落ちるのを確認した。
実害があったので直すことにした。

Go言語における最大値　符号付きint64　9223372036854775807 (2 ^ 63 - 1)

DBのSQLite3のint　　　INTEGERは、値に応じて 0～8byteの符号付き整数

ということで、データベースのアップグレード処理は書かなくても大丈夫だった。

![seed on bigint](https://github.com/pitapan5376/stable-diffusion-discord-bot/blob/master/document/008_seed_bigint.png?raw=true)


## 009. selection pop-up for sampler
you can select a sampler for image generation (optional, default is Euler_a)

sampler指定ができるようにした。未指定のときはEuler_aが指定されたものとみなす。

サンプラーの順番は自分で使いやすいように並べた。

![sampler_choice](https://github.com/pitapan5376/stable-diffusion-discord-bot/blob/master/document/009_sampler_selection.png?raw=true)

### Sampler: DPM++ S2 a Karras

` prompt: 1girl --seed 1 negative_prompt:EasyNegative sampler_name: DPM++ S2 a Karras `

![sampler_choice](https://github.com/pitapan5376/stable-diffusion-discord-bot/blob/master/document/009_sampler_DPMppS2aKarras.png?raw=true)

### Sampler: DPM Adaptive

` prompt: 1girl --seed 1 negative_prompt:EasyNegative sampler_name: DPM Adaptive `

![sampler_choice](https://github.com/pitapan5376/stable-diffusion-discord-bot/blob/master/document/009_sampler_DPMAdaptive.png?raw=true)

### Sampler: UniPC

` prompt: 1girl --seed 1 negative_prompt:EasyNegative sampler_name: UniPC `

![sampler_choice](https://github.com/pitapan5376/stable-diffusion-discord-bot/blob/master/document/009_sampler_UniPC.png?raw=true)

## 010. Hires.fix
hires.fix に部分的に対応した。
hr_scale（拡大率） とhr_upscaler（アップスケーラー名）をテーブルに追加した。
縦長などの時の破綻がなくなった。

実際の拡大率指定は未実装。

EXEを再実行するとき、SQLiteのマイグレーションで項目が追加される。
なので、それ以前の結果を選択して再生成・アップスケールを行うと、該当項目がないのでエラーになる。
imagineコマンドのリンクをクリックしてDBから再度プロンプトなどの内容を取得して、作り直せばよい。

強制的にHiresFixはONにしてある。
拡大率は2にしてあるが、--ar の指定で自動計算された縦横サイズのまま出力される。（そこから拡大はされない）
WebUIの動作を見ると拡大率を指定するか、または完成サイズを指定するようになっている。
元ソースの作りのまま、--ar 指定に応じて完成サイズを固定している。

## 011. BUGFIX: NegativePrompt
NegativePrompt didn't applied, have fixed it.
ネガティブプロンプトが反映されていなかったのに気付いたので確認して直した。


## 012. Hires.fix with Zoom Rate (param: --zoom)
to switch hires.fix ON/OFF with Discord choice way. default: OFF for better generation performance.
you can specify the ratio with '--zoom x.x' (default: 2.0) param only if hires_fix=YES

APIに渡すパラメータを修正して、hires.fix をオプションでON-OFF できるようにした。
![hiresfix1](https://github.com/pitapan5376/stable-diffusion-discord-bot/blob/master/document/012_hiresfix1.png?raw=true)

hires.fix のオプションをYESにしているとき、--zoom 1.2 のように指定すると元サイズから拡大される。

初期値は AUTO1111 WebUIにあわせて 2倍にした。

![hiresfix2](https://github.com/pitapan5376/stable-diffusion-discord-bot/blob/master/document/012_hiresfix2.png?raw=true)
![hiresfix4](https://github.com/pitapan5376/stable-diffusion-discord-bot/blob/master/document/imagine_20230624195929.jpg)

### hires.fix あり

![hiresfix3](https://github.com/pitapan5376/stable-diffusion-discord-bot/blob/master/document/012_hiresfix3.png?raw=true)

### hires.fix なし
hires.fix のオプションが未指定、またはNOのときは、zoom指定をしても無視される。

![hiresfix4](https://github.com/pitapan5376/stable-diffusion-discord-bot/blob/master/document/012_hiresfix4.png?raw=true)


## 013. apply upstream update
フォーク元が更新されていたので取り込んだ。バッチサイズが設定できるらしい。他に変更は特になし。（興味もない）

included upstream(AndBobsYourUncle's) update. It seems to change batch_size and program code improvements.
but I don't care about that.

## 014. BUTTON_COMPONENT_INVALID_EMOJI error on imagine_settings, new --px param will appended soon
an error calling imagine_settings command.
![invalid_emoji](https://github.com/pitapan5376/stable-diffusion-discord-bot/blob/master/document/014_emojierror.png?raw=true)

```
24/02/23 04:51:32 Error responding to interaction: HTTP 400 Bad Request, 
{"message": "Invalid Form Body", "code": 50035, "errors": {"data": 
{"components": {"0": {"components": {"0": {"options": {"0": 
{"emoji": {"name": {"_errors": [{"code": "BUTTON_COMPONENT_INVALID_EMOJI", "message": "Invalid emoji"}]}}}, 
"1": {"emoji": {"name": {"_errors": [{"code": "BUTTON_COMPONENT_INVALID_EMOJI", "message": "Invalid emoji"...
```
this seems in interaction between bot and Discord, but it's too difficult to solve for me.
in order to set pixel size, I will append `--px X,Y` param as in prompt option.

初期値設定コマンドが動作しないという報告を受けたので確認したところ絵文字に関するエラーが出ていた。
調べた範囲ではDiscordとのやり取りをするライブラリ内の問題らしく、修正は難しいと思われる。
サイズ変更のため、新たに `--px X,Y` の形式で指定可能にして、この後に`zoom`の適用ができるようにしておく予定。
（他オプションとの関連も検討してから近いうちに修正予定）

## 015. --px X,Y param
`--px X,Y`　の形式でベースの縦横幅を与えられるようにした。 `--ar`を指定してあっても、`--px`が優先される。
/imagine_settingsコマンドでの 512x512 や 768x768 は, --px を使うと意味をなさない。

ズーム指定とピクセル指定は同居できるので、 `--px 1000,200 --zoom 2` なら 2000x400の画像ができる。

区切り文字は半角コロン`:`またはカンマ`,`を受け付ける。後ろに空白があってもよい。

```
--px 1200:800
--px 1200,800
--px 1200, 800
```

最大サイズはメモリエラーが発生し得るので 8192 までにした。
メモリ不足エラーをつかむ方法はまだ分からないので自力で調整する必要がある。（他に何か起動していたらそちらで消費される）


I appended param as `--px X,Y` for base width and height. `--ar` is ignored if you state `--px` param.
/imagine_setting's default value is no longer effective if you use --px param.

you can use both --zoom and --px.
you would do as `1girl --px 1000:200 --zoom 2` then you get 2000x400 px.

the separator is accepted either colon or comma character, with and without space.

you could write as:

```
--px 1200:800
--px 1200,800
--px 1200, 800
```

Max. value of X,Y are 8192.

In this version, I cannot catch API Memory Out error so adjust yourself in base size.

### avoid timeout error with AUTO1111's API

巨大な画像を指定するとき、GPUメモリが足りているのに停止することがある。

AUTO1111のpythonWebサーバーのオプション `--timeout-keep-alive=` でタイムアウト値をのばすことで緩和は可能だが、
一定を超えるとDiscordとのやりとりでの進捗％の更新に失敗するようだ。

for large image, you could mitigate the timeout error which is between AUTO1111 WebUI API and program by `--timeout-keep-alive`,
but at some point of elasped processing time, interaction with Discord API will failed to update progress %.

Anyway, on AUTO1111's `webui-user.bat`, you could set timeout longer like this:

```
@echo off

set PYTHON=
set GIT=
set VENV_DIR=
set COMMANDLINE_ARGS= --api --nowebui --timeout-keep-alive=30000 --port 7860

call webui.bat

```

botコマンドのログ例

![bot_command_log_long_wait](https://github.com/pitapan5376/stable-diffusion-discord-bot/blob/master/document/015_bot_log.png?raw=true)

webuiのコンソール表示例

![web_ui_log_long_wait](https://github.com/pitapan5376/stable-diffusion-discord-bot/blob/master/document/015_webui-user_log.png?raw=true)


私の環境では2000x1000, 2000x512 は失敗し、 2000x256 (hires.fixで4000x512)は成功した。

In my environment, only 2000x256 was succeeded.(4000x512 on hires.fix with zoom 2) 

![015_timeout_discord](https://github.com/pitapan5376/stable-diffusion-discord-bot/blob/master/document/015_timeout_discord.png?raw=true)


![015_4000x512](https://github.com/pitapan5376/stable-diffusion-discord-bot/blob/master/document/015_4000x512.png)


## samples would be added here

あとでサンプル画像を足しておく。

I will add here sample generations.
 
