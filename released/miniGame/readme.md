# Mini game release

## 1. What is a mini game?

Mini games first came from WeChat, and the popularity of WeChat mini games began to drive other platforms to follow suit.

So what exactly are mini-games?

The official explanation of WeChat Mini Games is:

> [!Note]
>
> WeChat mini games are a category of WeChat mini programs. They are click-and-play, no need to download and install, and the experience is light. You can play with friends in WeChat, such as PK, watch, etc.

The core of its apparent experience is: no need to download and install, just click and play. As for the friend relationship chain, not all mini-games on all platforms are suitable for promotion using relationship chains.

> Other features will be introduced in other chapters.

What we need to know is that although most platforms continue the naming of WeChat mini games, also called XX mini games, there are also other names, such as Huawei and Xiaomi, both called XX quick games.

No matter what the name is, the basic experience and features of each platform are similar, so we can generally refer to these as small games.



## 2. Differences between mini games and HTML5

There is no need to download and install, just click and play. Isn’t it also possible with HTML5? How is it different from mini games?

### 2.1 Differences in running underlying layers

HTML5 links are usually run directly from the browser or webView.

When released into an APP, the LayaAir engine supports the self-developed Runtime underlying engine written in C++ to run the LayaAir engine project.

The principle of mini-games is similar to the mechanism of LayaAir releasing Native APP. It also uses the platform's built-in Runtime to be compatible with Canvas and WebGL interfaces, thereby achieving a click-and-play experience without downloading and installing HTML5.

Therefore, strictly speaking, mini-games are not HTML5 games because mini-games do not support all HTML5 standards and graphics API interfaces.

Of course, the LayaAir engine has been adapted to mainstream mini-game platforms. Developers using the LayaAir engine to develop projects can directly release them as mini-game products without modification.

### 2.2 Differences in package bodies

HTML5 resource packages and codes need to be loaded through the network before use.

The code package of the mini game must be placed on the mini game platform. By uploading and reviewing the code package of the mini game, the platform can strengthen the supervision of intellectual property rights and other benefits.

From a technical point of view, the size control of the mini-game code package and resource package allows developers to control the size of the project package according to the rules of the mini-game, ensuring that the game loading experience does not take too long.

### 2.3 Cache and other mini-game interfaces

In addition to the above introduction that is closely related to the game itself, in fact, the greater benefit of mini-games is that they provide more open interfaces than HTML5, such as more device interfaces, AI interfaces, payment interfaces, advertising interfaces, cache file management interfaces, Friend relationship link interface and so on.

### 2.4 Publishing template settings

Developers can create build-templates/platform names in the project directory, such as build-templates/web, etc., and the contents will be copied to the output directory when publishing. If it is a json file, it will also be merged. For example, if game.json is placed in build-templates/wxgame and the content is { "showStatusBar": true }, then this key value will be added to the final game.json file, and other key values ​​​​of game.json in the engine template Will keep it.



## 3. Which mini game platforms is LayaAir suitable for?

LayaAir 3.0 is currently adapted to the following platforms:

- #### [WeChat Mini Game](./wechat/readme.md)

- #### [Douyin Mini Game](./byteDance/readme.md)

- #### [OPPO Mini Game](./OPPO/readme.md)

- #### [vivo mini-game](./vivo/readme.md)

- #### [Xiaomi Quick Game](./xiaomi/readme.md)

- #### [Alipay mini-game](./alipaygame/readme.md)




