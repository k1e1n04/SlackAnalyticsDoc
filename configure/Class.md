# SlackAnalyticsAPIクラス図
## 注意
本クラス図はplantUMLを用いてVScodeにてプレビュー表示することを想定しております。  
以下の拡張機能VScodeにインストールしてこちらのファイルを開いてください。

- PlantUML
- Markdown Preview Enhanced

また下記のコマンドを用いてGraphvizをインストールしてください。(MacOSの場合)

    brew install graphviz

WindowsOSの場合はこちらからダウンロードできます。
[Graphviz](https://graphviz.gitlab.io/)

## クラス図
```plantuml
@startuml
class Organization {
    string name
    datetime created_at
    datetime updated_at
    string slack_app_token
}

class Base {
    string name
    datetime created_at
    datetime updated_at
    # organization[FK]
}


class Department {
    string name
    datetime created_at
    datetime updated_at
    # base [FK]
}

class Channel {
    string name
    string channel_id
    datetime created_at
    datetime updated_at
    # department [FK]
}

class Employee {
    string name
    string slack_id
    datetime created_at
    datetime updated_at
    # department [FK]
}

class Post {
    # channel[FK]
    # employee[FK]
    datetime created_at
}

class User {
    string name
    email
    # organization[FK]
    # base[FK]
    bool is_staff
    datetime created_at
}

Organization "1..*"--"*" Base
Base "1..*"--"*" Department
Department "1..*"--"*" Channel
Department "1..*"--"*" Employee 
Channel "1..*"--"*" Post
Employee "1..*"--"*" Post
Organization "1..*"--"*" User
Base "1..*"--"*" User




@enduml
```
