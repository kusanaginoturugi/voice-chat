# voice-chat

声で話しかけると、キャラクターが声で返事をする。全部自分のパソコンの中で動く。

```
声 → whisper-server（文字にする）→ llama-server（返事を考える）→ VOICEVOX（読み上げる）
```

- キーを押すと聞き始め、もう一度押すと返事をする。返事の途中で押すと、読み上げを止めて聞き始める。
- 返事は少しずつ受け取り、1 文できるたびに読み上げる。全部できあがるのを待たない。
- 「ずんだもんに切り替えて」「めたんに代わって」と言うと、話し相手が交代する。会話の流れは交代しても引き継ぐ。

## 使い方

```sh
voice-chat               # 1 回目で聞き始め、2 回目で返事をする（キーに割り当てる）
voice-chat --text 文章    # 声の代わりに文章で話しかける
voice-chat --file x.wav  # 音声ファイルで話しかける（16kHz モノラル）
voice-chat --reset       # 会話の履歴を消す
VOICE_CHAT_TIMING=1 voice-chat --text こんにちは  # 声が出るまでの時間を段階ごとに表示
```

## 必要なもの

- Python 3.11 以上（標準ライブラリだけで動く）
- whisper-server（whisper.cpp）、llama-server（llama.cpp）、VOICEVOX ENGINE
- PipeWire（pw-record と pw-play）

### VOICEVOX ENGINE を入れる

Docker で動かすのがいちばん手軽。NVIDIA の GPU があれば GPU 版にする（声を作る時間が CPU 版の 10 分の 1 ほどになり、VRAM は 770MiB ほど使う）。
コンテナから GPU を使うには nvidia-container-toolkit が要る。

```sh
sudo pacman -S docker nvidia-container-toolkit   # Arch 系
sudo nvidia-ctk runtime configure --runtime=docker
sudo systemctl enable --now docker && sudo systemctl restart docker

docker pull voicevox/voicevox_engine:nvidia-latest
docker run -d --name voicevox --restart unless-stopped --gpus all \
  -p '127.0.0.1:50021:50021' voicevox/voicevox_engine:nvidia-latest
curl http://127.0.0.1:50021/version   # 動いたか確かめる
```

GPU がなければ `voicevox/voicevox_engine:cpu-latest` を `--gpus all` なしで動かす。
Docker を使わない場合は、[公式リリース](https://github.com/VOICEVOX/voicevox_engine/releases/latest)からエンジンを落とすか、
[VOICEVOX](https://voicevox.hiroshiba.jp/) のエディタを起動しておく（起動中はエンジンも 50021 番で動く）。

つなぎ先とモデルは環境変数で変えられる。一覧は `voice-chat` の先頭のコメントに書いてある。

## VOICEVOX の VRAM を空ける（voicevox-trim）

VOICEVOX は、使った声のモデルを GPU に置いたままにし、一度確保した GPU のメモリも手放さない。
いろいろな声を使ったり歌わせたりすると、VRAM が 1〜3GB まで増える（起動した直後は 0.2GB ほど）。
`voicevox-trim` は、voicevox コンテナの VRAM が 1500MiB を超えていて、10 分リクエストがなければ、コンテナを再起動する。
再起動は数秒で終わる。

```sh
./deploy                              # voice-chat と voicevox-trim のリンクを置き、タイマーを有効にする（5 分おきに見る）
journalctl --user -u voicevox-trim    # 再起動したかどうか
```

`./deploy` は `~/.local/bin/voice-chat` と `~/.config/systemd/user/voicevox-trim.{service,timer}` を、リポジトリへのリンクとして置く。
配置先にリンクでない別のファイルがあると差分を出して止まり、`./deploy -f` で置き換える（元のファイルは `.bak` に残す）。

上限は `VOICEVOX_TRIM_MIB`、待つ時間は `VOICEVOX_TRIM_IDLE`（既定 `10m`）で変えられる。

## キャラクター

`characters.toml` に、名前・呼び名・VOICEVOX の声・話す速さ・人物像を書く。
足したキャラは、名前を呼べばすぐ切り替えられる。

VOICEVOX の各キャラクターの利用規約に従って使うこと。

## ライセンス

MIT。声とキャラクターの利用は、VOICEVOX と各キャラクターの利用規約に従う。
