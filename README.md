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

つなぎ先とモデルは環境変数で変えられる。一覧は `voice-chat` の先頭のコメントに書いてある。

## キャラクター

`characters.toml` に、名前・呼び名・VOICEVOX の声・話す速さ・人物像を書く。
足したキャラは、名前を呼べばすぐ切り替えられる。

VOICEVOX の各キャラクターの利用規約に従って使うこと。
