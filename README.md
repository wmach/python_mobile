# python でAndroidアプリを制作

画面ライブラリkivyで作成したGUIアプリを、buildozer でAndroid APK化

### 参考
- kivyのインストール
[kivy公式](https://kivy.org/doc/stable/gettingstarted/installation.html)
- buildozer github
[kivy/buildozer](https://hub.docker.com/r/kivy/buildozer)

### Windows11 で python と pip のアンインストール時に同時に削除するフォルダ

```
C:¥Users¥＜ユーザー名＞¥AppData¥Local¥Programs¥Python〇〇
C:¥Users¥＜ユーザー名＞¥AppData¥Local¥pipがある場合はフォルダごと削除
```

## 全体の流れ
1. windowsにkivyのサンプルをインストール
2. windowsで画面を表示
3. docker コンテナを作成
4. docker コンテナ内でkivyサンプルをAPKパッケージ化
5. APKパッケージを Android にインストールして画面を表示

## windowsにkivyのインストール

1. python3.8.10をインストール
[python公式](https://www.python.org/downloads/windows/)

2. pipとsetuptoolsの更新

```
py -m pip install -U pip setuptools
```

3. pip, setuptools, virtualenv の更新

```
py -m pip install --upgrade pip setuptools virtualenv
```

4. Virtual環境を作成

```
py -m virtualenv kivy_venv
```

5. 管理者としてPowershellを実行　※スクリプトが実行可能になっている場合は不要

6. スクリプトを実行可能に PowerShellから次を打鍵　※スクリプトが実行可能になっている場合は不要

```
Set-ExecutionPolicy RemoteSigned
```

7. Virtual環境を活性化

```
kivy_venv\Scripts\activate
```

6. kivy と kivyのサンプルのインストール

```
py -m pip install "kivy[base,audio]" kivy_examples
py -m pip install "kivy[base]" kivy_examples
```

8. サンプル起動

```
py kivy_venv\share\kivy-examples\demo\showcase\main.py
```

## buildozer の dockerコンテナを起動

1. 公式Dockerイメージクローン
[buildozer公式](git clone https://github.com/kivy/buildozer.git)

2. Dockerfile 編集

```
COPY requirements.txt requirements.txt
RUN pip3 install -r requirements.txt

# ENTRYPOINT ["buildozer"]
```

3. requirments.txt を作成して追加

```
buildozer
kivy
kivymd
pygments
```

4. docker-compose.yml を作成

```
version: '3'
services:
  buildozer:
    container_name: 'buildozer'
    build: .
    tty: true
    volumes:
      - type: bind
        source: C:\${自分のホームディレクトリ}\kivy_venv\share\kivy-examples\demo\showcase
        target: /home/user/hostcwd
      - type: bind
        source: C:\${buildozerのdockerイメージをcloneしたディレクトリ}
        target: /home/user/work
      - type: bind
        source: C:\${buildozerのdockerイメージをcloneしたディレクトリ}\..\buildozer_tool
        target: /home/user/.buildozer

```

5. docker containerを作成

```
docker-compose build
```

6. docker containerを起動

```
docker-compose up -d
```

7. docker containerにログイン

```
docker exec -it buildozer bash
```

## buildozerでkivyファイルをAndroidAPKにパッケージ化

1. buildozer を初期化

```
buildozer init
```

2. buildozer.spec ファイルを編集

'pygraments'を追加
```
requirements = python3,kivy,pygraments
```

3. buildozer で Android APK にパッケージ化

```
buildozer android debug
```

4. Android APK をインストール

```
adb devices
adb install C:${自分のホームディレクトリ}\kivy_venv\share\kivy-examples\demo\showcase\bin\myapp-0.1-arm64-v8a_armeabi-v7a-debug.apk
adb shell pm list packages
```

5. インストールしたAPKを起動

```
adb shell am start -n org.test.myapp/org.kivy.android.PythonActivity
```

