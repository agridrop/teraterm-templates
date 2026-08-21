# Tera Term Templates

Tera Term Macro（TTL）で利用できる、再利用可能なテンプレート集です。

SSH接続、セッションログ取得、接続先情報の記録、診断、定型操作など、Tera Termを利用した日常的な運用作業を効率化することを目的としています。

> [!NOTE]
> 本プロジェクトは非公式のコミュニティプロジェクトです。
> TeraTerm Projectとは提携・承認関係にありません。

## Features

現在、以下の用途を想定しています。

* SSH自動接続
* 公開鍵認証
* Pageantを利用した公開鍵認証
* パスワード認証
* SSHセッションログの自動保存
* SSH接続情報の記録
* 接続先のセッション情報取得
* `waitregex` を利用したプロンプト検出
* 接続エラー・タイムアウトの記録

今後、シリアル接続、ネットワーク機器操作、情報収集、設定バックアップなどのテンプレートも追加していく予定です。

## Templates

### `ssh-login.ttl`

SSH接続とセッションログ取得を行う汎用テンプレートです。

主な機能:

* IPv4 / IPv6 / FQDNによるSSH接続
* 複数のSSH認証方式
* 接続時のパスフレーズ・パスワード入力
* セッションごとのログファイル生成
* 接続情報のログ記録
* 接続先システム情報の自動取得
* プロンプト待機のタイムアウト検出
* マクロ実行後も継続した操作ログ取得
* エラー情報のCSV出力

## Requirements

* Tera Term
* Tera Term Macro

Tera Term 5系での利用を想定しています。

一部のマクロコマンドは古いバージョンのTera Termでは利用できない場合があります。

## Usage

### 1. テンプレートを取得

リポジトリをCloneするか、使用したいTTLファイルをダウンロードしてください。

```text
teraterm-templates/
├── README.md
├── LICENSE
└── templates/
    └── ssh-login.ttl
```

### 2. ユーザー設定領域を編集

`ssh-login.ttl` の先頭にある「ユーザー設定領域」を環境に合わせて変更します。

最低限、以下を設定してください。

```ttl
SSH_HOST = 'server.example.com'
SSH_PORT = 22
SSH_USER = 'username'

SSH_AUTH_MODE = 2
SSH_KEY_FILE = 'C:\Users\username\.ssh\id_ed25519'
```

原則として、「以下、原則変更不要」より後のコードを変更する必要はありません。

### 3. マクロを実行

Tera Term MacroからTTLファイルを実行してください。

正常に接続すると、SSHセッションが開始され、設定に応じてログファイルが生成されます。

## SSH Authentication

`SSH_AUTH_MODE` で認証方式を選択できます。

| Mode | 認証方式                   | 説明                  |
| ---: | ---------------------- | ------------------- |
|  `1` | Public Key / Automatic | 秘密鍵を直接使用して自動ログイン    |
|  `2` | Public Key / Prompt    | 秘密鍵を使用し、認証ダイアログを表示  |
|  `3` | Pageant                | Pageantに登録済みの秘密鍵を使用 |
|  `4` | Password / Prompt      | SSHパスワードを接続時に入力     |

デフォルトは `2` です。

```ttl
SSH_AUTH_MODE = 2
```

### Mode 1: Public Key / Automatic

```ttl
SSH_AUTH_MODE = 1
SSH_KEY_FILE = 'C:\path\to\privatekey'
```

主にパスフレーズなし秘密鍵を利用した自動ログイン向けです。

パスフレーズなし秘密鍵は、秘密鍵ファイルを取得された場合にそのまま悪用される可能性があります。ファイルへのアクセス権を適切に制限してください。

### Mode 2: Public Key / Prompt

```ttl
SSH_AUTH_MODE = 2
SSH_KEY_FILE = 'C:\path\to\privatekey'
```

公開鍵認証を使用し、接続時に認証ダイアログを表示します。

パスフレーズ付き秘密鍵の場合はパスフレーズを入力してください。

パスフレーズなし秘密鍵を使用する場合も利用でき、その場合は入力欄を空欄のまま進めることができます。

対話的にTera Termを利用する場合に扱いやすいため、本テンプレートではこの方式をデフォルトとしています。

### Mode 3: Pageant

```ttl
SSH_AUTH_MODE = 3
```

Pageantに登録済みの秘密鍵を利用します。

PageantはSSH Authentication Agentであり、OpenSSHの `ssh-agent` に相当する役割を持ちます。

秘密鍵をPageantへ登録しておくことで、Tera Termから秘密鍵ファイルを直接指定せずに認証できます。

### Mode 4: Password

```ttl
SSH_AUTH_MODE = 4
```

SSH接続時にパスワード入力ダイアログを表示します。

パスワードはTTLファイルには保存しません。

## Prompt Detection

SSH接続後のプロンプト検出には `waitregex` を使用しています。

デフォルトでは、一般的なLinux / Unix系シェルを想定しています。

```ttl
PROMPT_REGEX = '^[^\r\n]*[#$] *$'
```

以下のようなプロンプトに対応します。

```text
user@host:~$
root@host:~#
[user@host ~]$
```

ネットワーク機器などで `>` を利用する場合は、例えば以下のように変更してください。

```ttl
PROMPT_REGEX = '^[^\r\n]*[#$>] *$'
```

zsh等で `%` を利用する場合:

```ttl
PROMPT_REGEX = '^[^\r\n]*[#$>%] *$'
```

SSH接続自体には成功しているにもかかわらず `Prompt Timeout` が発生する場合は、最初に `PROMPT_REGEX` を確認してください。

## Session Logging

デフォルトではSSHセッションごとにログファイルを生成します。

```text
log-teraterm/
├── 20260822-001500_server.example.com_22_admin.log
├── 20260822-013025_192.168.1.10_22_root.log
└── _macro_error.csv
```

ログファイル名の形式:

```text
YYYYMMDD-HHMMSS_HOST_PORT_USER.log
```

日時を先頭にすることで、ファイル名による昇順ソートと時系列順が一致するようにしています。

### Logging Options

主要な設定値:

```ttl
LOG_ENABLE = 1
LOG_FILE_MODE = 1

LOG_BINARY = 0
LOG_APPEND = 1
LOG_PLAIN_TEXT = 0
LOG_TIMESTAMP = 1
LOG_HIDE_DIALOG = 1
LOG_INCLUDE_SCREEN_BUFFER = 0
LOG_TIMESTAMP_TYPE = 0

LOG_CONTINUE = 1
```

`LOG_CONTINUE = 1` の場合、マクロによる初期処理が終了した後もログ取得を継続します。

そのため、自動ログイン後にユーザーが実行した操作も同じログへ記録できます。

## Session Information

SSH接続直後に、以下の情報を自動取得できます。

| 情報                 | Command                  |
| ------------------ | ------------------------ |
| サーバ日時              | `date`                   |
| ホスト名               | `hostname`               |
| 実効ユーザー             | `whoami`                 |
| UID / GID / Groups | `id`                     |
| カレントディレクトリ         | `pwd`                    |
| Terminal / PTY     | `tty`                    |
| SSH接続情報            | `echo "$SSH_CONNECTION"` |

各項目は個別に有効・無効を設定できます。

```ttl
SESSION_INFO_DATE = 1
SESSION_INFO_HOST = 1
SESSION_INFO_USER = 1
SESSION_INFO_ID = 1
SESSION_INFO_WORKING_DIR = 1
SESSION_INFO_TTY = 1
SESSION_INFO_SSH_CONNECTION = 1
```

これらのコマンドは一般的なLinux / Unix系環境を想定しています。

ネットワーク機器や独自CLIなど、これらのコマンドが利用できない環境では以下を設定してください。

```ttl
SESSION_INFO_ENABLE = 0
```

## Nested SSH / `su` / `sudo`

Tera Termのセッションログは、端末へ送られてきた内容を継続して記録します。

そのため、SSH接続後に以下のような操作を行った場合も、表示された内容は同じログへ記録されます。

```text
user@server01:~$ su -
root@server01:~#

root@server01:~# ssh admin@server02
admin@server02:~$
```

ただし、ログヘッダに記録される以下の情報:

```text
SSH Host
SSH Port
SSH User
Authentication
```

は、**Tera Termが最初に直接確立したSSH接続**を示します。

`su`、`sudo`、Nested SSHによるユーザー・ホストの変更をTera Termが意味的に追跡して、ログヘッダを書き換えるわけではありません。

例えば、

```text
PC
 └─ SSH → server01 (user01)
          └─ su → root
                  └─ SSH → server02 (admin)
```

の場合でも、ログヘッダのSSH接続情報は `server01 / user01` のままです。

その後のユーザー・ホスト遷移については、端末ログそのものが証跡となります。

## Error Logging

マクロ内部で検出したエラーは、デフォルトでは以下へ追記されます。

```text
log-teraterm/_macro_error.csv
```

例:

```csv
"Timestamp","SSHHost","SSHPort","SSHUser","ErrorType"
"20260822-001500","server.example.com",22,"admin","SSH_CONNECT_FAILED"
"20260822-002030","server.example.com",22,"admin","PROMPT_TIMEOUT"
```

主なエラー種別:

* `SSH_CONNECT_FAILED`
* `SSH_KEY_FILE_NOT_FOUND`
* `SSH_KEY_FILE_IS_DIRECTORY`
* `LOG_OPEN_FAILED`
* `PROMPT_TIMEOUT`

## Security

### パスワード

SSHパスワードや秘密鍵のパスフレーズをTTLファイルへ直接保存する方式は採用していません。

対話認証ではTera Termの認証ダイアログを利用します。

### 秘密鍵

`SSH_AUTH_MODE = 1` でパスフレーズなし秘密鍵を利用する場合は、秘密鍵ファイルへのアクセス権を適切に制限してください。

対話利用では、パスフレーズ付き秘密鍵を利用する `SSH_AUTH_MODE = 2` やPageantの利用も検討してください。

### Host Key Verification

SSHホスト鍵の確認を無効化する設定は使用していません。

初回接続時などにTera Termからホスト鍵の確認を求められる場合があります。

安全性のため、接続先のホスト鍵を確認した上で登録してください。

## About Logging and Auditing

本プロジェクトで取得するログは、Tera Termクライアント側から見たSSHセッションログです。

サーバ側の以下のような監査ログを代替するものではありません。

* `sshd`
* `journald`
* `syslog`
* `auditd`
* Bastion / PAM / Session Recording製品等

厳密な操作監査が必要な環境では、サーバ側の監査ログと組み合わせて利用してください。

## Project Structure

現在は以下のような構成を想定しています。

```text
teraterm-templates/
├── README.md
├── LICENSE
└── templates/
    └── ssh-login.ttl
```

テンプレート数の増加に応じて、将来的には以下のような分類を検討しています。

```text
templates/
├── connection/
├── linux/
├── network/
├── transfer/
└── operations/
```

想定しているテンプレート例:

* SSH Login
* Jump Host
* Serial Console
* System Information Collection
* Log Collection
* Network Device Configuration Backup
* File Transfer
* Health Check
* Maintenance Operations

## Contributing

IssueやPull Requestによる改善提案・テンプレート追加を歓迎します。

テンプレートを追加する場合は、特定の環境へ過度に依存せず、設定値と内部処理を可能な限り分離することを推奨します。

また、パスワード、秘密鍵、内部IPアドレス、ホスト名などの機密情報をCommitしないよう注意してください。

## Disclaimer

本プロジェクトで提供されるテンプレートは、利用者自身の責任において使用してください。

本番環境へ適用する前に、対象環境で十分に動作確認することを推奨します。

特に、自動化されたコマンド実行やネットワーク機器の設定変更を行うテンプレートについては、実行内容を確認してから使用してください。

## License

This project is licensed under the Apache License 2.0.

詳細は [LICENSE](LICENSE) を参照してください。

## Acknowledgements

Tera TermおよびTera Term MacroはTeraTerm Projectによって開発・提供されています。

本プロジェクトはTeraTerm Projectとは独立した非公式プロジェクトです。
