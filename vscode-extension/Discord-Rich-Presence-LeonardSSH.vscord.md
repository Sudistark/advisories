
My friends are always active on Discord and they tend use extensions which integerates with Dicord Rich Presence to have some cool status on their Discord profile.

A common example would be this extension which are available for Vscode:

https://marketplace.visualstudio.com/items?itemName=LeonardSSH.vscord ( 250,319 installs )
https://marketplace.visualstudio.com/items?itemName=icrawl.discord-vscode ( 1,329,713 installs )

From the number of installs you can see such extensions are really very popuplar. You can find similar extension for other applications also like Spotify,Valorant,Genshin Impact,etc


To start using this extension you just need to install it in Vscode and make sure you are using Discord Desktop app,the communication b/w this extension and Discord app happens through  websocket.
Once the extension is installed just start working on you repository.

Here's how it appears on the Discord UI (both vscode extension have the same view as you can see in this screenshot):
![image](https://user-images.githubusercontent.com/31372554/233820334-462e19c2-2660-4ef6-913d-d0f4e6376899.png)

If you click on the *View Repository* button, this popup will appear:

![image](https://user-images.githubusercontent.com/31372554/233820391-ca93d949-364c-4956-aca7-cc492251634b.png)


If you proceed to click on *Yep!*, it will open the repository url in your browser.

------------------

From my understanding the `{git_url}` placeholder value is taken from the .git/config file

```
[core]
        repositoryformatversion = 0
        filemode = false
        bare = false
        logallrefupdates = true
        ignorecase = true
[remote "origin"]
        url = https://github.com/Sudistark/test-discord-vscode
        fetch = +refs/heads/*:refs/remotes/origin/*
[branch "master"]
        remote = origin
        merge = refs/heads/master
```

Whatever is in the url is used, passed to Discord and is viewable to other Discord users also upon clicking on the *View Repository* button.

Most users would be signed in to their GitHub acc in vscode, so there is no requirement for a GitHub personal access token.


But suppose you are not logged in so upon trying to clone a private repo or making commits would ask you for GitHub credentials

A prompt like this appear:
![chrome_qxNwX7MrYQ](https://user-images.githubusercontent.com/31372554/233851141-99ae2082-2fdf-48a7-bd25-a5625d79beb7.png)



username is your GitHub username and for password you need to use your GitHub personal access token.
This username password will stored in .git/config file like this:

```
[core]
        repositoryformatversion = 0
        filemode = false
        bare = false
        logallrefupdates = true
        ignorecase = true
[remote "origin"]
        url = https://sudistark:ghp_xxxxxxxxxxxxxxx@github.com/Sudistark/test-discord-vscode
        fetch = +refs/heads/*:refs/remotes/origin/*
[branch "master"]
        remote = origin
        merge = refs/heads/master
```

Noticed the `@` part before github.com contains your credentials.

![image](https://user-images.githubusercontent.com/31372554/233851285-6a946116-e23e-470b-895c-4979072199ba.png)


-------------

The extension had no checks for git_url , even when the url contained the credentials it was pass as it is to Discord due to which any other Discord user would've have got the user's GitHub personal access token by copying  the Repository url.

Browsers hides the credentials part ,for example if you were to visit this url suppose in Google Chrome :

https://sudistark:ghp_xxxxxxxx@github.com/sudistark/private-repo

In the address bar it would have appear like this instead:
https://github.com/sudistark/private-repo

It was  visible like this only (the credentials part was hidden ) even in Discord when you clicked on the *View Repository* button.

As the credentials part was hidden all along in Discord and even when you clicked on *View Repository* button, nobody would have been able to figured it out that there's a bug which is exposing the user's GitHub Personal access token to the public.

Someone  would have been only able to identify this bug when he opens your discord profile, clicks on the *View Repository* button then again clicks on the *Yep* button confirming to open it in browser  and then copies the url from the browser address bar.

---------------------

I found this bug totally by mistake, I was just causally checking what my developer friend was doing, so I copied the repo url and pasted in our Discord chat and there I saw a GitHub token.

I had no fucking idea from where that token came from, at first I had doubt that this token is mine but when I checked it with one of the api calls. I saw my friend's username there in the response which confirmed that it was his token.

In the past I have copied my friend's repo url many times but never saw the GitHub token before so I asked him about it to identify the cause of this bug, he told me that he cloned the repo from his another acc using the GitHub token this time.

After trying the same thing I was able to reproduce this behaviour.

----------------------

I quickly forwarded the details to the Project Maintainer , which were very swift in responding and addressing the issue: https://github.com/leonardssh/vscord/issues/209

A special shoutouts to @leonardssh and @xhayper, I really love when secuirty reports are handle like this.
@xhayper noticed that another extension which had the same function was also vulnerable so he came out with a fix for it also, tryly awesome work.

The fix was pretty simple: https://github.com/leonardssh/vscord/commit/a1a1d51ae4415584ab2c6d6fe9a7ac5a0cdd85d8

```js
import stripCredential from "./helpers/stripCredential";


    public get gitRemoteUrl(): gitUrlParse.GitUrl | undefined {
        const v = stripCredential(this._remote?.fetchUrl ?? this._remote?.pushUrl ?? "");
        this.debug(`gitRemoteUrl(): Url: ${v ?? ""}`);
        if (!v) return;

        return gitUrlParse(v);
    }
```

https://github.com/leonardssh/vscord/blob/a1a1d51ae4415584ab2c6d6fe9a7ac5a0cdd85d8/src/helpers/stripCredential.ts

```js
import { URL } from "node:url";

export default function (uri: string): string {
    try {
        const url = new URL(uri);
        url.username = "";
        url.password = "";
        return url.toString();
    } catch (ignored) {
        return uri;
    }
}
```

Now the credential part is removed from the url before it is passed to Discord.

--------------

If you are interested in reproducing the bug, then just downgrade your extension it should be below 5.1.8

**Steps to reproduce**

1. Make sure you aren't signed in to your Github acc in Vscode (instead we will rely upon the Username/Password based authentication for github )
2. Create any private repository in your github acc
3. Then clone it in your pc in vscode window (as we are not authenticated a prompt like this will appear)
![chrome_qxNwX7MrYQ](https://user-images.githubusercontent.com/31372554/233614351-27c74855-ca99-4ec7-a20f-7cd7a82acb0c.png)

4. Fill in the username and your github personal token as your password

https://docs.github.com/en/authentication/keeping-your-account-and-data-secure/creating-a-personal-access-token

5. Now start editing any file in that repo and check discord
6. Click on the View Repository button
7. Click on Yep to open the url in browser
8. From the Browser address bar , copy the url there you will notice that you have got your github token also.

You can verify your token from here

```
curl -s -u "user:ghp_************" https://api.github.com/user
```
