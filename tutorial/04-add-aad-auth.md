<!-- markdownlint-disable MD002 MD041 -->

<span data-ttu-id="680ff-101">Neste exercício, você estenderá o aplicativo do exercício anterior para dar suporte à autenticação com o Azure AD.</span><span class="sxs-lookup"><span data-stu-id="680ff-101">In this exercise you will extend the application from the previous exercise to support authentication with Azure AD.</span></span> <span data-ttu-id="680ff-102">Isso é necessário para obter o token de acesso OAuth necessário para chamar o Microsoft Graph.</span><span class="sxs-lookup"><span data-stu-id="680ff-102">This is required to obtain the necessary OAuth access token to call the Microsoft Graph.</span></span> <span data-ttu-id="680ff-103">Para fazer isso, você integrará a Biblioteca de Autenticação da [Microsoft (MSAL) para iOS](https://github.com/AzureAD/microsoft-authentication-library-for-objc) ao aplicativo.</span><span class="sxs-lookup"><span data-stu-id="680ff-103">To do this, you will integrate the [Microsoft Authentication Library (MSAL) for iOS](https://github.com/AzureAD/microsoft-authentication-library-for-objc) into the application.</span></span>

1. <span data-ttu-id="680ff-104">Crie um novo **arquivo de lista de** propriedades no projeto **GraphTutorial** chamado **AuthSettings.plist**.</span><span class="sxs-lookup"><span data-stu-id="680ff-104">Create a new **Property List** file in the **GraphTutorial** project named **AuthSettings.plist**.</span></span>
1. <span data-ttu-id="680ff-105">Adicione os itens a seguir ao arquivo no **dicionário** raiz.</span><span class="sxs-lookup"><span data-stu-id="680ff-105">Add the following items to the file in the **Root** dictionary.</span></span>

    | <span data-ttu-id="680ff-106">Chave</span><span class="sxs-lookup"><span data-stu-id="680ff-106">Key</span></span> | <span data-ttu-id="680ff-107">Tipo</span><span class="sxs-lookup"><span data-stu-id="680ff-107">Type</span></span> | <span data-ttu-id="680ff-108">Valor</span><span class="sxs-lookup"><span data-stu-id="680ff-108">Value</span></span> |
    |-----|------|-------|
    | `AppId` | <span data-ttu-id="680ff-109">Cadeia de caracteres</span><span class="sxs-lookup"><span data-stu-id="680ff-109">String</span></span> | <span data-ttu-id="680ff-110">A ID do aplicativo do portal do Azure</span><span class="sxs-lookup"><span data-stu-id="680ff-110">The application ID from the Azure portal</span></span> |
    | `GraphScopes` | <span data-ttu-id="680ff-111">Matriz</span><span class="sxs-lookup"><span data-stu-id="680ff-111">Array</span></span> | <span data-ttu-id="680ff-112">Três valores de cadeia de `User.Read` caracteres: `MailboxSettings.Read` , e `Calendars.ReadWrite`</span><span class="sxs-lookup"><span data-stu-id="680ff-112">Three String values: `User.Read`, `MailboxSettings.Read`, and `Calendars.ReadWrite`</span></span> |

    ![Captura de tela do arquivo AuthSettings.plist no Xcode](images/auth-settings.png)

> [!IMPORTANT]
> <span data-ttu-id="680ff-114">Se você estiver usando o controle de código-fonte, como git, agora seria um bom momento para excluir o arquivo **AuthSettings.plist** do controle de código-fonte para evitar o vazamento inadvertida da ID do aplicativo.</span><span class="sxs-lookup"><span data-stu-id="680ff-114">If you're using source control such as git, now would be a good time to exclude the **AuthSettings.plist** file from source control to avoid inadvertently leaking your app ID.</span></span>

## <a name="implement-sign-in"></a><span data-ttu-id="680ff-115">Implementar a login</span><span class="sxs-lookup"><span data-stu-id="680ff-115">Implement sign-in</span></span>

<span data-ttu-id="680ff-116">Nesta seção, você configurará o projeto para MSAL, criará uma classe de gerenciador de autenticação e atualizará o aplicativo para entrar e sair.</span><span class="sxs-lookup"><span data-stu-id="680ff-116">In this section you will configure the project for MSAL, create an authentication manager class, and update the app to sign in and sign out.</span></span>

### <a name="configure-project-for-msal"></a><span data-ttu-id="680ff-117">Configurar o projeto para MSAL</span><span class="sxs-lookup"><span data-stu-id="680ff-117">Configure project for MSAL</span></span>

1. <span data-ttu-id="680ff-118">Adicione um novo grupo de chaves aos recursos do seu projeto.</span><span class="sxs-lookup"><span data-stu-id="680ff-118">Add a new keychain group to your project's capabilities.</span></span>
    1. <span data-ttu-id="680ff-119">Selecione o **projeto GraphTutorial** e, em seguida, **assinando & recursos.**</span><span class="sxs-lookup"><span data-stu-id="680ff-119">Select the **GraphTutorial** project, then **Signing & Capabilities**.</span></span>
    1. <span data-ttu-id="680ff-120">Select **+ Capability**, then double-click **Keychain Sharing**.</span><span class="sxs-lookup"><span data-stu-id="680ff-120">Select **+ Capability**, then double-click **Keychain Sharing**.</span></span>
    1. <span data-ttu-id="680ff-121">Adicione um grupo de chaves com o `com.microsoft.adalcache` valor.</span><span class="sxs-lookup"><span data-stu-id="680ff-121">Add a keychain group with the value `com.microsoft.adalcache`.</span></span>

1. <span data-ttu-id="680ff-122">Controle clique **em Info.plist** e selecione **Abrir como**, em seguida, **Código-fonte**.</span><span class="sxs-lookup"><span data-stu-id="680ff-122">Control click **Info.plist** and select **Open As**, then **Source Code**.</span></span>
1. <span data-ttu-id="680ff-123">Adicione o seguinte dentro do `<dict>` elemento.</span><span class="sxs-lookup"><span data-stu-id="680ff-123">Add the following inside the `<dict>` element.</span></span>

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

1. <span data-ttu-id="680ff-124">Abra **AppDelegate.swift** e adicione a seguinte instrução de importação na parte superior do arquivo.</span><span class="sxs-lookup"><span data-stu-id="680ff-124">Open **AppDelegate.swift** and add the following import statement at the top of the file.</span></span>

    ```Swift
    import MSAL
    ```

1. <span data-ttu-id="680ff-125">Adicione a função a seguir à classe `AppDelegate`.</span><span class="sxs-lookup"><span data-stu-id="680ff-125">Add the following function to the `AppDelegate` class.</span></span>

    :::code language="swift" source="../demo/GraphTutorial/GraphTutorial/AppDelegate.swift" id="HandleMsalResponseSnippet":::

### <a name="create-authentication-manager"></a><span data-ttu-id="680ff-126">Criar gerenciador de autenticação</span><span class="sxs-lookup"><span data-stu-id="680ff-126">Create authentication manager</span></span>

1. <span data-ttu-id="680ff-127">Crie um novo **Arquivo Swift no** projeto **GraphTutorial** chamado **AuthenticationManager.swift.**</span><span class="sxs-lookup"><span data-stu-id="680ff-127">Create a new **Swift File** in the **GraphTutorial** project named **AuthenticationManager.swift**.</span></span> <span data-ttu-id="680ff-128">Adicione o código a seguir a esse arquivo.</span><span class="sxs-lookup"><span data-stu-id="680ff-128">Add the following code to the file.</span></span>

    :::code language="swift" source="../demo/GraphTutorial/GraphTutorial/AuthenticationManager.swift" id="AuthManagerSnippet":::

### <a name="add-sign-in-and-sign-out"></a><span data-ttu-id="680ff-129">Adicionar entrar e sair</span><span class="sxs-lookup"><span data-stu-id="680ff-129">Add sign-in and sign-out</span></span>

1. <span data-ttu-id="680ff-130">Abra **SignInViewController.swift e** substitua seu conteúdo pelo código a seguir.</span><span class="sxs-lookup"><span data-stu-id="680ff-130">Open **SignInViewController.swift** and replace its contents with the following code.</span></span>

    :::code language="swift" source="../demo/GraphTutorial/GraphTutorial/SignInViewController.swift" id="SignInViewSnippet":::

1. <span data-ttu-id="680ff-131">Abra **WelcomeViewController.swift** e substitua a função `signOut` existente pelo seguinte.</span><span class="sxs-lookup"><span data-stu-id="680ff-131">Open **WelcomeViewController.swift** and replace the existing `signOut` function with the following.</span></span>

    :::code language="swift" source="../demo/GraphTutorial/GraphTutorial/WelcomeViewController.swift" id="SignOutSnippet":::

1. <span data-ttu-id="680ff-132">Salve suas alterações e reinicie o aplicativo no Simulador.</span><span class="sxs-lookup"><span data-stu-id="680ff-132">Save your changes and restart the application in Simulator.</span></span>

<span data-ttu-id="680ff-133">Se você entrar no aplicativo, deverá ver um token de acesso exibido na janela de saída no Xcode.</span><span class="sxs-lookup"><span data-stu-id="680ff-133">If you sign in to the app, you should see an access token displayed in the output window in Xcode.</span></span>

![Uma captura de tela da janela de saída no Xcode mostrando um token de acesso](images/access-token-output.png)

## <a name="get-user-details"></a><span data-ttu-id="680ff-135">Obter detalhes do usuário</span><span class="sxs-lookup"><span data-stu-id="680ff-135">Get user details</span></span>

<span data-ttu-id="680ff-136">Nesta seção, você criará uma classe auxiliar para manter todas as chamadas para o Microsoft Graph e atualizar para usar essa nova classe para obter o usuário `WelcomeViewController` conectado.</span><span class="sxs-lookup"><span data-stu-id="680ff-136">In this section you will create a helper class to hold all of the calls to Microsoft Graph and update the `WelcomeViewController` to use this new class to get the logged-in user.</span></span>

1. <span data-ttu-id="680ff-137">Crie um novo **Arquivo Swift no** projeto **GraphTutorial** chamado **GraphManager.swift.**</span><span class="sxs-lookup"><span data-stu-id="680ff-137">Create a new **Swift File** in the **GraphTutorial** project named **GraphManager.swift**.</span></span> <span data-ttu-id="680ff-138">Adicione o código a seguir a esse arquivo.</span><span class="sxs-lookup"><span data-stu-id="680ff-138">Add the following code to the file.</span></span>

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

1. <span data-ttu-id="680ff-139">Abra **WelcomeViewController.swift** e adicione a `import` instrução a seguir na parte superior do arquivo.</span><span class="sxs-lookup"><span data-stu-id="680ff-139">Open **WelcomeViewController.swift** and add the following `import` statement at the top of the file.</span></span>

    ```Swift
    import MSGraphClientModels
    ```

1. <span data-ttu-id="680ff-140">Adicione a propriedade a seguir à `WelcomeViewController` classe.</span><span class="sxs-lookup"><span data-stu-id="680ff-140">Add the following property to the `WelcomeViewController` class.</span></span>

    ```Swift
    private let spinner = SpinnerViewController()
    ```

1. <span data-ttu-id="680ff-141">Substitua o existente `viewDidLoad` pelo código a seguir.</span><span class="sxs-lookup"><span data-stu-id="680ff-141">Replace the existing `viewDidLoad` with the following code.</span></span>

    :::code language="swift" source="../demo/GraphTutorial/GraphTutorial/WelcomeViewController.swift" id="ViewDidLoadSnippet":::

<span data-ttu-id="680ff-142">Se você salvar suas alterações e reiniciar o aplicativo agora, depois de entrar na interface do usuário for atualizado com o nome de exibição e o endereço de email do usuário.</span><span class="sxs-lookup"><span data-stu-id="680ff-142">If you save your changes and restart the app now, after sign-in the UI is updated with the user's display name and email address.</span></span>
