[![Open In Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/github/Soliton-Analytics-Team/ParlAI-Japanese/blob/master/ParlAI.ipynb)

# ColabでFacebookのSOTAなチャットボットを動かして、ついでに日本語化してみた

[こんな記事](https://ai.facebook.com/blog/state-of-the-art-open-source-chatbot)をみました。すごく自然な対話ができるということでColabで試しました。以下の手順で実行するだけで再現できるようにしてあります。

さっそく[github](https://github.com/facebookresearch/ParlAI)の手順に従って、インストールします。


```Shell
!git clone https://github.com/facebookresearch/ParlAI.git /root/ParlAI
%cd /root/ParlAI
!python setup.py develop
```

チャットボットを走らせる前に以下のコマンドを実行する必要があります。これをせずに走らせるとエラーとなって、このコマンドを先に実行せよとのメッセージが出ます。ここではあらかじめ実行しておきましょう。


```Shell
!pip install 'git+https://github.com/rsennrich/subword-nmt.git#egg=subword-nmt'
```

下記のコマンドで一番小さいサイズのチャットボットが走ります。初回起動時は90Mバイトのモデルをダウンロードするので、入力状態になるまで数十秒の時間がかかります。「Enter Your Message:」のプロンプトが出たら、英語でメッセージを入力してみましょう。対話を終了するには[EXIT]を入力します。


```Shell
!python parlai/scripts/safe_interactive.py -t blended_skill_talk -mf zoo:blender/blender_90M/model
```

以下に実行例を示します。
<pre>
[context]: your persona: i have bowled a 300 game twice in my life.
your persona: i have been held hostage in a store robbery situation.
Enter Your Message: Hello
[TransformerGenerator]: hi ! how are you ? i ' m a yoga instructor . what do you like to do ?
Enter Your Message: I like to play tennis.
[TransformerGenerator]: that ' s cool . i like watching sports . do you have any hobbies besides tennis ?
Enter Your Message: No.
[TransformerGenerator]: tennis is a great sport . i love watching it . what ' s your favorite sport to play ?
Enter Your Message: Tennis!
[TransformerGenerator]: i ' ve never played tennis , but i ' d love to try . what else do you do for fun ?
Enter Your Message: Nothing.
[TransformerGenerator]: have you ever tried yoga ? it ' s a great way to relax and get to know yourself .
Enter Your Message: No, I have not.
[TransformerGenerator]: you should give it a try . it can be a lot of work , but it is very relaxing .
Enter Your Message: Good.
[TransformerGenerator]: what is your favorite type of yoga ? mine is hatha yoga , which i like a lot .
Enter Your Message: I don't know.
[TransformerGenerator]: well , you should try it . it will help you get used to the different types of yoga .
Enter Your Message: [EXIT]
CHAT DONE.
Your partner was playing the following persona:
partner's persona: watching sports is fun.
partner's persona: i go to yoga.
EPOCH DONE
</pre>
意外と簡単に動きました。ちぐはぐなところもありますが、一番小さいモデルなので、仕方ありません。ペルソナがボーリング好きに設定されているようなので、ボーリングの話をするべきだったのかもしれません。反応速度は悪くありませんが、CPUのみで動かす場合は、このモデルが限界のようです。

中規模のモデルで動かしてみましょう。初回起動時は3Gバイトのモデルをダウンロードするので、入力状態になるまで十分程度の時間がかかります。



```
!python parlai/scripts/safe_interactive.py -t blended_skill_talk -mf zoo:blender/blender_3B/model
```

<pre>
[context]: your persona: i'm an avid quilter.
your persona: my grand children mean the world to me.
Enter Your Message: Hello
[TransformerGenerator]: How are you doing this evening? I'm just relaxing in my car. It's a 20 year old car.
Enter Your Message: I'm fine. Is it a classic car?
[TransformerGenerator]: Yes, it's a 1965 Chevrolet Camaro. What do you drive?
Enter Your Message: I drive a BMW.
[TransformerGenerator]: That's nice. Do you have any hobbies that you like to do in your spare time?
Enter Your Message: My hobby is tennis.
[TransformerGenerator]: Tennis is fun. I used to play a lot when I was younger. What is your favorite color?
Enter Your Message: I love red!
[TransformerGenerator]: That's a great color. I like red too. What else do you enjoy doing in your free time?
Enter Your Message: I enjoy reading books.
[TransformerGenerator]: Reading is a great way to pass the time. What are some of your favorite authors?
Enter Your Message: I love Osamu Dazai.
[TransformerGenerator]: I've never heard of him, but I'll have to check him out. What genre of books does he write?
Enter Your Message: [EXIT]
CHAT DONE.
Your partner was playing the following persona:
partner's persona: my car is over 20 years old.
partner's persona: i did not finish college.
EPOCH DONE
</pre>
スムーズな会話ができてます。ColabでGPUをオンにすれば、反応速度もまずまずで実用に足ります。CPUのみの場合は何分も応答が返って来ないので、GPUは必須です。よく見ると、応答がちゃんとキャピタライズされています。最小モデルではそうなっていないので、そのあたりの品質も向上するようです。



最大モデルはblender_9Bなのですが、これは起動すらしません。少なくとも32GBのV100 GPUが二つ必要とのことなので、Colabでは動きません。

ところで、上記のコマンドsafe_interactive.pyのsafeとはなんでしょう。実は暴言をチェックして警告する機能が付いているのです。これを外すにはinteractive.pyを用います。また以降のsafe_local_humanもlocal_humanに置き換えれば暴言チェックなしになります。

さて、これだけだと、ただのご紹介になってしまうので、これを[translators](https://github.com/uliontse/translators)を使って日本語化してみましょう。入力と出力にGoogle翻訳を挟みます。translatorsのインストールと使い方は以下になります。ただし、利用は学生および個人に制限されていますので、商用にはご使用にならないよう、お願いします。


```
!pip install translators
```


```Python
import translators as ts
from loguru import logger
logger.disable('translators') # ログ出力を抑制します
print(ts.google("How are you doing this evening? I'm just relaxing in my car. It's a 20 year old car.", to_language='ja'))
```

    今夜はどうですか？車でくつろぐだけです。 20歳の車です。


Google翻訳、いまいちですね。実はこれ、DeepL翻訳だと「今日の夜はどうですか？私は車でのんびりしています。20年前の車です。」と翻訳されてはるかに良いのですが、残念ながらtranslatorsのDeepL翻訳だとすぐにtoo many requestsのエラーが出てしまいます。

デバッガーでステップ実行して調べたところ、ユーザーからの入力は以下の部分でした。


```
!cat -n parlai/agents/safe_local_human/safe_local_human.py | tail +97 | head -5
```

        97	    def get_reply(self):
        98	        reply_text = input(colorize('Enter Your Message:', 'field') + ' ')
        99	        reply_text = reply_text.replace('\\n', '\n')
       100	
       101	        return reply_text


これを以下のコマンドで書き換えます。


```
!mv parlai/agents/safe_local_human/safe_local_human.py .
!sed -e "12i import translators as ts" \
 -e "s/return reply_text$/reply_text = ts.google(reply_text, to_language='en'); print(reply_text); return reply_text/" \
 ./safe_local_human.py > parlai/agents/safe_local_human/safe_local_human.py
!cat -n parlai/agents/safe_local_human/safe_local_human.py | tail +98 | head -5
```

    /root/ParlAI
        98	    def get_reply(self):
        99	        reply_text = input(colorize('Enter Your Message:', 'field') + ' ')
       100	        reply_text = reply_text.replace('\\n', '\n')
       101	
       102	        reply_text = ts.google(reply_text, to_language='en'); print(reply_text); return reply_text


チャットボットから出力は以下の部分です。


```
!cat -n parlai/utils/misc.py | tail +793 | head -2
```

       793	        if msg.get('text', ''):
       794	            text = clip_text(msg['text'], max_len)


これを以下のコマンドで書き換えます


```
!mv parlai/utils/misc.py .
!sed -e "10i import translators as ts" \
  -e "s/], max_len)$/], max_len); print(text); text = ts.google(text, to_language='ja')/" \
  ./misc.py > parlai/utils/misc.py
!cat -n parlai/utils/misc.py | tail +794 | head -2
```

    /root/ParlAI
       794	        if msg.get('text', ''):
       795	            text = clip_text(msg['text'], max_len); print(text); text = ts.google(text, to_language='ja')


それでは実行してみましょう。翻訳前後のテキストも表示するようにしてあります。


```
%cd /root/ParlAI
!python parlai/scripts/safe_interactive.py -t blended_skill_talk -mf zoo:blender/blender_3B/model
```

<pre>
Enter Your Message: こんにちは
Hello
How are you doing this evening? I'm just hanging out with my friend, who lives in Japan.
[TransformerGenerator]: 今夜はどうですか？私は日本に住んでいる私の友人とたむろしています。
Enter Your Message: 自分も日本に住んでいます。
I also live in Japan.
What do you like to do for fun? I like to go to the arcade and play video games.
[TransformerGenerator]: 楽しみのために何をしたいですか？私はアーケードに行き、ビデオゲームをするのが好きです。
Enter Your Message: 私はテニスをするのが好きです。
I like playing tennis
Tennis is a lot of fun. Do you have a favorite color? Mine is blue.
[TransformerGenerator]: テニスはとても楽しいです。好きな色はありますか？鉱山は青です。
Enter Your Message: 私は赤が好きです。
I like red
Red is a nice color. I like blue because it reminds me of the ocean. Where do you live?
[TransformerGenerator]: 赤はいい色です。海を思わせる青が好きです。どこに住んでいますか？
Enter Your Message: 東京に住んでいます。
I live in Tokyo.
I've never been there, but I've always wanted to go. What is your favorite thing to do there?
[TransformerGenerator]: 行ったことはありませんが、ずっと行きたかったです。そこで好きなことは何ですか？
Enter Your Message: 食べ歩きが好きです。
I like eating and walking.
That sounds like a great way to spend a day. I would love to go there some day.
[TransformerGenerator]: それは1日を過ごすのに最適な方法のように思えます。いつか行ってみたいです。
Enter Your Message: ぜひ来てください。
Please come.
[ Sorry, could not process that message. Please try again. ]
Enter Your Message: ぜひ観光に来てください。
Please come visit us.
I'd love to! I'll have to save up some money first. What do you do for a living?
[TransformerGenerator]: 是非！最初にお金を貯めなければならない。どんなお仕事をしていらっしゃいますか？
Enter Your Message: 私はエンジニアです。
I am an engineer.
That's cool. I work at a grocery store as a cashier. It pays the bills.
[TransformerGenerator]: カッコいい。私はレジで食料品店で働いています。手形を支払う。
Enter Your Message: 仕事は大変ですか？
Is it hard to work?
It can be at times, but it's not too bad. What kind of engineer are you?
[TransformerGenerator]: 時々あるかもしれませんが、それほど悪くはありません。あなたはどんなエンジニアですか？
Enter Your Message: プログラミングの仕事です。
It's a programming job.
[ Sorry, could not process that message. Please try again. ]
Enter Your Message: プログラムをする仕事です。
It is a job of programming.
That's awesome. I wish I had more time to learn how to program. Do you have any hobbies?
[TransformerGenerator]: それは素晴らしいです。プログラミングの方法を学ぶ時間がもっとあればいいのですが。趣味がありますか？
Enter Your Message: もう行かないと
I have to go
I'm sorry to hear that. What are you going to do while you're gone? I like to read.
[TransformerGenerator]: 申し訳ありません。行っている間に何をするつもりですか？私は読むのが好き。
Enter Your Message: さよなら
Goodbye
I hope you have a good rest of your day. Thanks for chatting with me. Take care.
[TransformerGenerator]: 私はあなたが一日の残りを過ごすことを望みます。私とチャットしてくれてありがとう。気を付けて。
Enter Your Message: [EXIT]
[EXIT]

CHAT DONE.

Your partner was playing the following persona:
partner's persona: i live in new york.
partner's persona: my best friend lives in japan.
EPOCH DONE
</pre>

ところどころ翻訳がダメなところがあったり、検閲に引っかかったりしてます。でも、会話自体はまあまあ成り立ってます。

いかがでしたでしょう。意外と簡単に日本語化できましたね。
