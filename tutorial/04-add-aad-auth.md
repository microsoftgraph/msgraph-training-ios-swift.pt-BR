<!-- markdownlint-disable MD002 MD041 -->

Neste exercício, você estenderá o aplicativo do exercício anterior para dar suporte à autenticação com o Azure AD. Isso é necessário para obter o token de acesso OAuth necessário para chamar o Microsoft Graph. Para fazer isso, você integrará a Biblioteca de Autenticação da [Microsoft (MSAL) para iOS](https://github.com/AzureAD/microsoft-authentication-library-for-objc) ao aplicativo.

1. Crie um novo **arquivo de lista de** propriedades no projeto **GraphTutorial** chamado **AuthSettings.plist**.
1. Adicione os itens a seguir ao arquivo no **dicionário** raiz.

    | Chave | Tipo | Valor |
    |-----|------|-------|
    | `AppId` | Cadeia de caracteres | A ID do aplicativo do portal do Azure |
    | `GraphScopes` | Matriz | Três valores de cadeia de `User.Read` caracteres: `MailboxSettings.Read` , e `Calendars.ReadWrite` |

    ![Captura de tela do arquivo AuthSettings.plist no Xcode](images/auth-settings.png)

> [!IMPORTANT]
> Se você estiver usando o controle de código-fonte, como git, agora seria um bom momento para excluir o arquivo **AuthSettings.plist** do controle de código-fonte para evitar o vazamento inadvertida da ID do aplicativo.

## <a name="implement-sign-in"></a>Implementar a login

Nesta seção, você configurará o projeto para MSAL, criará uma classe de gerenciador de autenticação e atualizará o aplicativo para entrar e sair.

### <a name="configure-project-for-msal"></a>Configurar o projeto para MSAL

1. Adicione um novo grupo de chaves aos recursos do seu projeto.
    1. Selecione o **projeto GraphTutorial** e, em seguida, **assinando & recursos.**
    1. Select **+ Capability**, then double-click **Keychain Sharing**.
    1. Adicione um grupo de chaves com o `com.microsoft.adalcache` valor.

1. Controle clique **em Info.plist** e selecione **Abrir como**, em seguida, **Código-fonte**.
1. Adicione o seguinte dentro do `<dict>` elemento.

    ```xml
    <key>CFBundleURLTypes</key>
    <array>
      <dict>
        <key>CFBundleURLSchemes</key>
        <array>
          <string>msauth.$(PRODUCT_BUNDLE_IDENTIFIER)</string>
        </array>
      </dict>
    </array>
    <key>LSApplicationQueriesSchemes</key>
    <array>
        <string>msauthv2</string>
        <string>msauthv3</string>
    </array>
    ```

1. Abra **AppDelegate.swift** e adicione a seguinte instrução de importação na parte superior do arquivo.

    ```Swift
    import MSAL
    ```

1. Adicione a função a seguir à classe `AppDelegate`.

    :::code language="swift" source="../demo/GraphTutorial/GraphTutorial/AppDelegate.swift" id="HandleMsalResponseSnippet":::

### <a name="create-authentication-manager"></a>Criar gerenciador de autenticação

1. Crie um novo **Arquivo Swift no** projeto **GraphTutorial** chamado **AuthenticationManager.swift.** Adicione o código a seguir a esse arquivo.

    :::code language="swift" source="../demo/GraphTutorial/GraphTutorial/AuthenticationManager.swift" id="AuthManagerSnippet":::

### <a name="add-sign-in-and-sign-out"></a>Adicionar entrar e sair

1. Abra **SignInViewController.swift e** substitua seu conteúdo pelo código a seguir.

    :::code language="swift" source="../demo/GraphTutorial/GraphTutorial/SignInViewController.swift" id="SignInViewSnippet":::

1. Abra **WelcomeViewController.swift** e substitua a função `signOut` existente pelo seguinte.

    :::code language="swift" source="../demo/GraphTutorial/GraphTutorial/WelcomeViewController.swift" id="SignOutSnippet":::

1. Salve suas alterações e reinicie o aplicativo no Simulador.

Se você entrar no aplicativo, deverá ver um token de acesso exibido na janela de saída no Xcode.

![Uma captura de tela da janela de saída no Xcode mostrando um token de acesso](images/access-token-output.png)

## <a name="get-user-details"></a>Obter detalhes do usuário

Nesta seção, você criará uma classe auxiliar para manter todas as chamadas para o Microsoft Graph e atualizar para usar essa nova classe para obter o usuário `WelcomeViewController` conectado.

1. Crie um novo **Arquivo Swift no** projeto **GraphTutorial** chamado **GraphManager.swift.** Adicione o código a seguir a esse arquivo.

    ```Swift
    import Foundation
    import MSGraphClientSDK
    import MSGraphClientModels

    class GraphManager {

        // Implement singleton pattern
        static let instance = GraphManager()

        private let client: MSHTTPClient?

        public var userTimeZone: String

        private init() {
            client = MSClientFactory.createHTTPClient(with: AuthenticationManager.instance)
            userTimeZone = "UTC"
        }

        public func getMe(completion: @escaping(MSGraphUser?, Error?) -> Void) {
            // GET /me
            let select = "$select=displayName,mail,mailboxSettings,userPrincipalName"
            let meRequest = NSMutableURLRequest(url: URL(string: "\(MSGraphBaseURL)/me?\(select)")!)
            let meDataTask = MSURLSessionDataTask(request: meRequest, client: self.client, completion: {
                (data: Data?, response: URLResponse?, graphError: Error?) in
                guard let meData = data, graphError == nil else {
                    completion(nil, graphError)
                    return
                }

                do {
                    // Deserialize response as a user
                    let user = try MSGraphUser(data: meData)
                    completion(user, nil)
                } catch {
                    completion(nil, error)
                }
            })

            // Execute the request
            meDataTask?.execute()
        }
    }
    ```

1. Abra **WelcomeViewController.swift** e adicione a `import` instrução a seguir na parte superior do arquivo.

    ```Swift
    import MSGraphClientModels
    ```

1. Adicione a propriedade a seguir à `WelcomeViewController` classe.

    ```Swift
    private let spinner = SpinnerViewController()
    ```

1. Substitua o existente `viewDidLoad` pelo código a seguir.

    :::code language="swift" source="../demo/GraphTutorial/GraphTutorial/WelcomeViewController.swift" id="ViewDidLoadSnippet":::

Se você salvar suas alterações e reiniciar o aplicativo agora, depois de entrar na interface do usuário for atualizado com o nome de exibição e o endereço de email do usuário.
