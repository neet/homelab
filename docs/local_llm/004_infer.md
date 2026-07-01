# とりあえず推論してみる

uvを使いたいので入れる。アンインストールの仕方は[Installation | uv](https://docs.astral.sh/uv/getting-started/installation/)にある。

```
curl -LsSf https://astral.sh/uv/install.sh | sh
```

実験用のリポを [neetlab/playground-cuda-language-model](https://github.com/neetlab/playground-cuda-language-model) に作ったので、以下で試す。

```
uv sync
uv run main.py
```

CUDAがPyTorchに認識されているか試す。

```
neet@compute-mitaka-02:~/playground-cuda-language-model$ uv run python
Python 3.13.14 (main, Jun 23 2026, 15:18:27) [Clang 22.1.3 ] on linux
Type "help", "copyright", "credits" or "license" for more information.
>>> import torch
>>> torch.cuda.is_available()
True
```

すんなり動いてくれた。CUDAツールキットみたいなやつは必要なかったのかな？

```
[{'generated_text': "Hello! My name is David, and this is my story. I'm an indie game developer from Seattle, and I was one of the first to make a Kickstarter campaign. I've been working on indie games since I was 12, so I've been working there for the last eight years. I've been taking a lot of different risks, which in and of itself is really amazing.\n\nI've worked very hard to make the game you see here. There's a lot to learn about the games you've played, from the lore, the art, the sound, the environments and the story, to the design and the writing and the sound. To me, that's the hardest part. I've been trying to figure out what makes a good game, and I've really been working to that goal.\n\nI do want to thank you for your time and support. I'm looking forward to getting back to you soon, so stay tuned for more information about the project.\n\nIf you enjoyed this interview, please support the Indie Game Spotlight on Patreon.\n\nThe Indie Game Spotlight is a weekly series examining the best games for the indie genre. Each week, we'll be talking about something new—some games we think are great for the indie community, some we think aren"}, {'generated_text': "Hello! My name is Hilda and I'm a member of the World of Warcraft guild! I'm a master of the Arcane spellcasting, combat magic, and elemental magic. I'm also a member of the guild of the Living Evil, who have been fighting through a lot of grief for the last few months. My guild is based in the Netherlands, but I'm not quite there yet. I've been working hard to prepare for the new year, but my time is up this summer so hopefully the new year will be a good one for me.\n\nYou can check out your awesome friends on here (that's me in the photo!)."}, {'generated_text': "Hello! My name is Jia, and I'm a pretty sweet, sexy dude. I'm also a bit of an asshole, so I didn't feel comfortable talking with you guys through Skype. So... I'm here to talk to you guys! I'm very serious about this, but I'm going to show you some of my best and most beautiful pictures! First off, it's not a picture I'm going to show you. I'm not sure if you know what I mean. So, first off, let me say how amazing it is to see you here at this place. The only thing I'm not going to show you is your date. You know what I mean? I told you I was cool, you know what I mean? So, first off, let me just say that I'm a big fan of girl fans and I love what they do. I love the kind of girl fans that they get. I'm a huge fan of her, and I love how she likes to be around me. She has a really great time with her, and she's really, really nice. So, first off, let me just say that I get a lot of great questions about girls online, and I'm sure that I've got a lot of them. You know"}, {'generated_text': 'Hello! My name is Mimi and you\'re welcome to join me in making this one a reality! Please let me know if you\'d like to join me."\n\n"Ah… yeah."\n\nMimi frowned and put her hand on the table. "This is a new place, it\'s a new place. They\'re just going to make me stay for a while, so you\'ll be there to stay here. And this is where it gets weird, it\'s not a place where you can just come and go. I don\'t want to be like that, I want to be like this place. I want to stay here, and I want to be like this place, and I want to be like this place, and I want to make you stay here, and I want to make you stay here, and I want to make you stay here, and I want to make you stay here, and I want to make you stay here, and I want to make you stay here, and I want to make you stay here, and I want to make you stay here, and I want to make you stay here, and I want to make you stay here, and I want to make you stay here, and I want to make you stay here, and I want to'}, {'generated_text': "Hello! My name is Tommi. I am a girl in my late teens. I love anime. I love making new friends, I love reading books, and I love making people smile. The most important thing for me right now is to start my life anew and work to get to where I am today. I always think about the best way to start my life, but I have a lot of feelings for the people that came before me. I also think that I am a good person. I love people. I love being a human being. I love living and working with people. I love being a human being.I feel that I am really good at everything. I feel that I am really good at everything. I believe that I can be anything in this world. This is my world, and I can do anything. I hope to become like you by becoming your own person.I'm trying to live a life of pure love. I love living and working. I love making new friends and making people smile. I love making people smile. I love doing things that I can't do. I love making people smile.I love the things that I can't do. I love being a human being. I'm doing everything, and I'm happy to be living my life"}]
```

この記事を参考にして、推論時のモニタリングをしてみた。うまく動いていそう：[nvidia-smi #GPU - Qiita](https://qiita.com/OV104/items/ecad7f8842a82e0c679b#3-%E8%A9%B3%E7%B4%B0%E6%83%85%E5%A0%B1%E3%81%AE%E7%85%A7%E4%BC%9Aquery-%E3%82%AA%E3%83%97%E3%82%B7%E3%83%A7%E3%83%B3%E3%81%A7%E3%83%94%E3%83%B3%E3%83%9D%E3%82%A4%E3%83%B3%E3%83%88%E3%81%AB)

```
neet@compute-mitaka-02:~$ nvidia-smi --query-gpu=timestamp,utilization.gpu,memory.used,temperature.gpu --format=csv,noheader -l 1
2026/07/01 20:02:28.692, 0 %, 1 MiB, 43
2026/07/01 20:02:29.700, 0 %, 1 MiB, 43
2026/07/01 20:02:30.700, 0 %, 1 MiB, 43
2026/07/01 20:02:31.700, 0 %, 1 MiB, 43
2026/07/01 20:02:32.701, 0 %, 1 MiB, 43
2026/07/01 20:02:33.701, 0 %, 1 MiB, 43
2026/07/01 20:02:34.701, 11 %, 1036 MiB, 46
2026/07/01 20:02:35.701, 96 %, 1676 MiB, 56
2026/07/01 20:02:36.702, 0 %, 1 MiB, 50
2026/07/01 20:02:37.702, 0 %, 1 MiB, 50
2026/07/01 20:02:38.702, 0 %, 1 MiB, 48
2026/07/01 20:02:39.703, 0 %, 1 MiB, 47
```
