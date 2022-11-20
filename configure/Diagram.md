# SlackAnalytics構成図
## 注意
本構成図はplantUMLを用いて作成しております。  
### Choromeで閲覧する場合
拡張機能をインストールしてください。  
- [Pegmatite - Chrome ウェブストア](https://chrome.google.com/webstore/detail/pegmatite/jegkfbnfbfnohncpcfcimepibmhlkldo)

### VScodeで閲覧する場合
以下の拡張機能VScodeにインストールしてこちらのファイルを開いてください。

- PlantUML
- Markdown Preview Enhanced

また下記のコマンドを用いてGraphvizをインストールしてください。(MacOSの場合)

    brew install graphviz

WindowsOSの場合はこちらからダウンロードできます。
[Graphviz](https://graphviz.gitlab.io/)

## 構成図
```plantuml
@startuml
folder "Client" {
    [PC]
}

cloud "Azure Container Apps"{
    node "nginx"
    node "SlackAnalyticsBackEnd(DjangoRESTAPI)"
    node "SlackAnalyticsFrontEnd(React)"
    node {
        database "MySQL"{
            [slackanalytics]
        }
    }
    node "PostAnalyticsServer"
}
folder SlackAPI

[PC]-->[nginx]:HTTP Req
[PC]<--[nginx]:HTTP Res
[nginx]-->[SlackAnalyticsFrontEnd(React)]:Port Forward
[nginx]<--[SlackAnalyticsFrontEnd(React)]:Res
[nginx]-->[SlackAnalyticsBackEnd(DjangoRESTAPI)]:Port Forward
[nginx]<--[SlackAnalyticsBackEnd(DjangoRESTAPI)]:Res
[SlackAnalyticsFrontEnd(React)]-->[SlackAnalyticsBackEnd(DjangoRESTAPI)]:HPPT Req
[SlackAnalyticsFrontEnd(React)]<--[SlackAnalyticsBackEnd(DjangoRESTAPI)]:HPPT Res
[SlackAnalyticsBackEnd(DjangoRESTAPI)]-->[slackanalytics]:SQL(select)
[SlackAnalyticsBackEnd(DjangoRESTAPI)]<--[slackanalytics]:All Tables Data
[PostAnalyticsServer]-->[slackanalytics]: SQL(Post Table Data(create))
[PostAnalyticsServer]<--[slackanalytics]: Organization,Channel,Employee Table Data
[PostAnalyticsServer]-->[SlackAPI]:HTTP Req
[PostAnalyticsServer]<--[SlackAPI]:HTTP Res

@enduml
```
