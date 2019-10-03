<!-- markdownlint-disable MD002 MD041 -->

<span data-ttu-id="b25e7-101">Neste exercício, você estenderá o aplicativo do exercício anterior para oferecer suporte à autenticação com o Azure AD.</span><span class="sxs-lookup"><span data-stu-id="b25e7-101">In this exercise you will extend the application from the previous exercise to support authentication with Azure AD.</span></span> <span data-ttu-id="b25e7-102">Isso é necessário para obter o token de acesso OAuth necessário para chamar o Microsoft Graph.</span><span class="sxs-lookup"><span data-stu-id="b25e7-102">This is required to obtain the necessary OAuth access token to call the Microsoft Graph.</span></span> <span data-ttu-id="b25e7-103">Para fazer isso, você integrará a [biblioteca de autenticação da Microsoft (MSAL) para IOS](https://github.com/AzureAD/microsoft-authentication-library-for-objc) no aplicativo.</span><span class="sxs-lookup"><span data-stu-id="b25e7-103">To do this, you will integrate the [Microsoft Authentication Library (MSAL) for iOS](https://github.com/AzureAD/microsoft-authentication-library-for-objc) into the application.</span></span>

1. <span data-ttu-id="b25e7-104">Crie um novo arquivo de **lista de propriedades** no projeto do **GraphTutorial** chamado **AuthSettings. plist**.</span><span class="sxs-lookup"><span data-stu-id="b25e7-104">Create a new **Property List** file in the **GraphTutorial** project named **AuthSettings.plist**.</span></span>
1. <span data-ttu-id="b25e7-105">Adicione os seguintes itens ao arquivo no dicionário **raiz** .</span><span class="sxs-lookup"><span data-stu-id="b25e7-105">Add the following items to the file in the **Root** dictionary.</span></span>

    | <span data-ttu-id="b25e7-106">Chave</span><span class="sxs-lookup"><span data-stu-id="b25e7-106">Key</span></span> | <span data-ttu-id="b25e7-107">Tipo</span><span class="sxs-lookup"><span data-stu-id="b25e7-107">Type</span></span> | <span data-ttu-id="b25e7-108">Valor</span><span class="sxs-lookup"><span data-stu-id="b25e7-108">Value</span></span> |
    |-----|------|-------|
    | `AppId` | <span data-ttu-id="b25e7-109">String</span><span class="sxs-lookup"><span data-stu-id="b25e7-109">String</span></span> | <span data-ttu-id="b25e7-110">A ID do aplicativo do portal do Azure</span><span class="sxs-lookup"><span data-stu-id="b25e7-110">The application ID from the Azure portal</span></span> |
    | `GraphScopes` | <span data-ttu-id="b25e7-111">Matriz</span><span class="sxs-lookup"><span data-stu-id="b25e7-111">Array</span></span> | <span data-ttu-id="b25e7-112">Dois valores de cadeia `User.Read` de caracteres: e`Calendars.Read`</span><span class="sxs-lookup"><span data-stu-id="b25e7-112">Two String values: `User.Read` and `Calendars.Read`</span></span> |

    ![Uma captura de tela do arquivo AuthSettings. plist no Xcode](./images/auth-settings.png)

> [!IMPORTANT]
> <span data-ttu-id="b25e7-114">Se você estiver usando o controle de origem como o Git, agora seria uma boa hora para excluir o arquivo **AuthSettings. plist** do controle de origem para evitar vazar inadvertidamente sua ID de aplicativo.</span><span class="sxs-lookup"><span data-stu-id="b25e7-114">If you're using source control such as git, now would be a good time to exclude the **AuthSettings.plist** file from source control to avoid inadvertently leaking your app ID.</span></span>

## <a name="implement-sign-in"></a><span data-ttu-id="b25e7-115">Implementar logon</span><span class="sxs-lookup"><span data-stu-id="b25e7-115">Implement sign-in</span></span>

<span data-ttu-id="b25e7-116">Nesta seção, você irá configurar o projeto para o MSAL, criar uma classe do Gerenciador de autenticação e atualizar o aplicativo para entrar e sair.</span><span class="sxs-lookup"><span data-stu-id="b25e7-116">In this section you will configure the project for MSAL, create an authentication manager class, and update the app to sign in and sign out.</span></span>

### <a name="configure-project-for-msal"></a><span data-ttu-id="b25e7-117">Configurar o Project para o MSAL</span><span class="sxs-lookup"><span data-stu-id="b25e7-117">Configure project for MSAL</span></span>

1. <span data-ttu-id="b25e7-118">Control clique **info. plist** e selecione **abrir como**e, em seguida, **código-fonte**.</span><span class="sxs-lookup"><span data-stu-id="b25e7-118">Control click **Info.plist** and select **Open As**, then **Source Code**.</span></span>
1. <span data-ttu-id="b25e7-119">Adicione o seguinte no `<dict>` elemento.</span><span class="sxs-lookup"><span data-stu-id="b25e7-119">Add the following inside the `<dict>` element.</span></span>

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
    <array>
        <string>msauthv2</string>
        <string>msauthv3</string>
    </array>
    ```

1. <span data-ttu-id="b25e7-120">Abra **AppDelegate. Swift** e adicione a seguinte instrução de importação na parte superior do arquivo.</span><span class="sxs-lookup"><span data-stu-id="b25e7-120">Open **AppDelegate.swift** and add the following import statement at the top of the file.</span></span>

    ```Swift
    import MSAL
    ```

1. <span data-ttu-id="b25e7-121">Adicione a função a seguir à classe `AppDelegate`.</span><span class="sxs-lookup"><span data-stu-id="b25e7-121">Add the following function to the `AppDelegate` class.</span></span>

    ```Swift
    func application(_ app: UIApplication, open url: URL, options: [UIApplication.OpenURLOptionsKey : Any] = [:]) -> Bool {

        guard let sourceApplication = options[UIApplication.OpenURLOptionsKey.sourceApplication] as? String else {
            return false
        }

        return MSALPublicClientApplication.handleMSALResponse(url, sourceApplication: sourceApplication)
    }
    ```

### <a name="create-authentication-manager"></a><span data-ttu-id="b25e7-122">Criar o Gerenciador de autenticação</span><span class="sxs-lookup"><span data-stu-id="b25e7-122">Create authentication manager</span></span>

1. <span data-ttu-id="b25e7-123">Crie um novo **arquivo Swift** no projeto **GraphTutorial** chamado **AuthenticationManager. Swift**.</span><span class="sxs-lookup"><span data-stu-id="b25e7-123">Create a new **Swift File** in the **GraphTutorial** project named **AuthenticationManager.swift**.</span></span> <span data-ttu-id="b25e7-124">Adicione o código a seguir a esse arquivo.</span><span class="sxs-lookup"><span data-stu-id="b25e7-124">Add the following code to the file.</span></span>

    ```Swift
    import Foundation
    import MSAL

    class AuthenticationManager {

        // Implement singleton pattern
        static let instance = AuthenticationManager()

        private let publicClient: MSALPublicClientApplication?
        private let appId: String
        private let graphScopes: Array<String>

        private init() {
            // Get app ID and scopes from AuthSettings.plist
            let bundle = Bundle.main
            let authConfigPath = bundle.path(forResource: "AuthSettings", ofType: "plist")!
            let authConfig = NSDictionary(contentsOfFile: authConfigPath)!

            self.appId = authConfig["AppId"] as! String
            self.graphScopes = authConfig["GraphScopes"] as! Array<String>

            do {
                // Create the MSAL client
                try self.publicClient = MSALPublicClientApplication(clientId: self.appId,
                                                                    keychainGroup: bundle.bundleIdentifier)
            } catch {
                print("Error creating MSAL public client: \(error)")
                self.publicClient = nil
            }
        }

        public func getPublicClient() -> MSALPublicClientApplication? {
            return self.publicClient
        }

        public func getTokenInteractively(completion: @escaping(_ accessToken: String?, Error?) -> Void) {
            // Call acquireToken to open a browser so the user can sign in
            publicClient?.acquireToken(forScopes: self.graphScopes, completionBlock: {
                (result: MSALResult?, error: Error?) in
                guard let tokenResult = result, error == nil else {
                    print("Error getting token interactively: \(String(describing: error))")
                    completion(nil, error)
                    return
                }

                print("Got token interactively: \(tokenResult.accessToken)")
                completion(tokenResult.accessToken, nil)
            })
        }

        public func getTokenSilently(completion: @escaping(_ accessToken: String?, Error?) -> Void) {
            // Check if there is an account in the cache
            var userAccount: MSALAccount?

            do {
                userAccount = try publicClient?.allAccounts().first
            } catch {
                print("Error getting account: \(error)")
            }

            if (userAccount != nil) {
                // Attempt to get token silently
                publicClient?.acquireTokenSilent(forScopes: self.graphScopes, account: userAccount!, completionBlock: {
                    (result: MSALResult?, error: Error?) in
                    guard let tokenResult = result, error == nil else {
                        print("Error getting token silently: \(String(describing: error))")
                        completion(nil, error)
                        return
                    }

                    print("Got token silently: \(tokenResult.accessToken)")
                    completion(tokenResult.accessToken, nil)
                })
            } else {
                print("No account in cache")
                completion(nil, NSError(domain: "AuthenticationManager",
                                        code: MSALErrorCode.interactionRequired.rawValue, userInfo: nil))
            }
        }

        public func signOut() -> Void {
            do {
                // Remove all accounts from the cache
                let accounts = try publicClient?.allAccounts()

                try accounts!.forEach({
                    (account: MSALAccount) in
                    try publicClient?.remove(account)
                })
            } catch {
                print("Sign out error: \(String(describing: error))")
            }
        }
    }
    ```

### <a name="add-sign-in-and-sign-out"></a><span data-ttu-id="b25e7-125">Adicionar entrada e saída</span><span class="sxs-lookup"><span data-stu-id="b25e7-125">Add sign-in and sign-out</span></span>

1. <span data-ttu-id="b25e7-126">Abra **SignInViewController. Swift** e substitua seu conteúdo pelo código a seguir.</span><span class="sxs-lookup"><span data-stu-id="b25e7-126">Open **SignInViewController.swift** and replace its contents with the following code.</span></span>

    ```Swift
    import UIKit

    class SignInViewController: UIViewController {

        private let spinner = SpinnerViewController()

        override func viewDidLoad() {
            super.viewDidLoad()
            // Do any additional setup after loading the view.

            // See if a user is already signed in
            spinner.start(container: self)

            AuthenticationManager.instance.getTokenSilently {
                (token: String?, error: Error?) in

                DispatchQueue.main.async {
                    self.spinner.stop()

                    guard let _ = token, error == nil else {
                        // If there is no token or if there's an error,
                        // no user is signed in, so stay here
                        return
                    }

                    // Since we got a token, a user is signed in
                    // Go to welcome page
                    self.performSegue(withIdentifier: "userSignedIn", sender: nil)
                }
            }
        }

        @IBAction func signIn() {
            spinner.start(container: self)

            // Do an interactive sign in
            AuthenticationManager.instance.getTokenInteractively {
                (token: String?, error: Error?) in

                DispatchQueue.main.async {
                    self.spinner.stop()

                    guard let _ = token, error == nil else {
                        // Show the error and stay on the sign-in page
                        let alert = UIAlertController(title: "Error signing in",
                                                      message: error.debugDescription,
                                                      preferredStyle: .alert)

                        alert.addAction(UIAlertAction(title: "OK", style: .default, handler: nil))
                        self.present(alert, animated: true)
                        return
                    }

                    // Signed in successfully
                    // Go to welcome page
                    self.performSegue(withIdentifier: "userSignedIn", sender: nil)
                }
            }
        }
    }
    ```

1. <span data-ttu-id="b25e7-127">Abra **WelcomeViewController. Swift** e substitua a função `signOut` existente pelo seguinte.</span><span class="sxs-lookup"><span data-stu-id="b25e7-127">Open **WelcomeViewController.swift** and replace the existing `signOut` function with the following.</span></span>

    ```Swift
    @IBAction func signOut() {
        AuthenticationManager.instance.signOut()
        self.performSegue(withIdentifier: "userSignedOut", sender: nil)
    }
    ```

1. <span data-ttu-id="b25e7-128">Salve suas alterações e reinicie o aplicativo no simulador.</span><span class="sxs-lookup"><span data-stu-id="b25e7-128">Save your changes and restart the application in Simulator.</span></span>

<span data-ttu-id="b25e7-129">Se você entrar no aplicativo, verá um token de acesso exibido na janela de saída do Xcode.</span><span class="sxs-lookup"><span data-stu-id="b25e7-129">If you sign in to the app, you should see an access token displayed in the output window in Xcode.</span></span>

![Uma captura de tela da janela de saída no Xcode mostrando um token de acesso](./images/access-token-output.png)

## <a name="get-user-details"></a><span data-ttu-id="b25e7-131">Obter detalhes do usuário</span><span class="sxs-lookup"><span data-stu-id="b25e7-131">Get user details</span></span>

<span data-ttu-id="b25e7-132">Nesta seção, você criará uma classe auxiliar para manter todas as chamadas para o Microsoft Graph e atualizará `WelcomeViewController` o para usar essa nova classe para obter o usuário conectado.</span><span class="sxs-lookup"><span data-stu-id="b25e7-132">In this section you will create a helper class to hold all of the calls to Microsoft Graph and update the `WelcomeViewController` to use this new class to get the logged-in user.</span></span>

1. <span data-ttu-id="b25e7-133">Abra **AuthenticationManager. Swift** e adicione a função a seguir à `AuthenticationManager` classe.</span><span class="sxs-lookup"><span data-stu-id="b25e7-133">Open **AuthenticationManager.swift** and add the following function to the `AuthenticationManager` class.</span></span>

    ```Swift
    public func getGraphAuthProvider() -> MSALAuthenticationProvider? {
        // Create an MSAL auth provider for use with the Graph client
        return MSALAuthenticationProvider(publicClientApplication: self.publicClient,
                                          andScopes: self.graphScopes)
    }
    ```

1. <span data-ttu-id="b25e7-134">Crie um novo **arquivo Swift** no projeto **GraphTutorial** chamado **graphmanager. Swift**.</span><span class="sxs-lookup"><span data-stu-id="b25e7-134">Create a new **Swift File** in the **GraphTutorial** project named **GraphManager.swift**.</span></span> <span data-ttu-id="b25e7-135">Adicione o código a seguir a esse arquivo.</span><span class="sxs-lookup"><span data-stu-id="b25e7-135">Add the following code to the file.</span></span>

    ```Swift
    import Foundation
    import MSGraphClientSDK
    import MSGraphClientModels

    class GraphManager {

        // Implement singleton pattern
        static let instance = GraphManager()

        private let client: MSHTTPClient?

        private init() {
            client = MSClientFactory.createHTTPClient(with: AuthenticationManager.instance.getGraphAuthProvider())
        }

        public func getMe(completion: @escaping(MSGraphUser?, Error?) -> Void) {
            // GET /me
            let meRequest = NSMutableURLRequest(url: URL(string: "\(MSGraphBaseURL)/me")!)
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

1. <span data-ttu-id="b25e7-136">Abra **WelcomeViewController. Swift** e adicione a seguinte `import` instrução na parte superior do arquivo.</span><span class="sxs-lookup"><span data-stu-id="b25e7-136">Open **WelcomeViewController.swift** and add the following `import` statement at the top of the file.</span></span>

    ```Swift
    import MSGraphClientModels
    ```

1. <span data-ttu-id="b25e7-137">Adicione a propriedade a seguir à `WelcomeViewController` classe.</span><span class="sxs-lookup"><span data-stu-id="b25e7-137">Add the following property to the `WelcomeViewController` class.</span></span>

    ```Swift
    private let spinner = SpinnerViewController()
    ```

1. <span data-ttu-id="b25e7-138">Substitua o existente `viewDidLoad` pelo código a seguir.</span><span class="sxs-lookup"><span data-stu-id="b25e7-138">Replace the existing `viewDidLoad` with the following code.</span></span>

    ```Swift
    override func viewDidLoad() {
        super.viewDidLoad()

        // Do any additional setup after loading the view.

        self.spinner.start(container: self)

        // Get the signed-in user
        self.userProfilePhoto.image = UIImage(imageLiteralResourceName: "DefaultUserPhoto")

        GraphManager.instance.getMe {
            (user: MSGraphUser?, error: Error?) in

            DispatchQueue.main.async {
                self.spinner.stop()

                guard let currentUser = user, error == nil else {
                    print("Error getting user: \(String(describing: error))")
                    return
                }

                // Set display name
                self.userDisplayName.text = currentUser.displayName ?? "Mysterious Stranger"
                self.userDisplayName.sizeToFit()

                // AAD users have email in the mail attribute
                // Personal accounts have email in the userPrincipalName attribute
                self.userEmail.text = currentUser.mail ?? currentUser.userPrincipalName ?? ""
                self.userEmail.sizeToFit()
            }
        }
    }
    ```

<span data-ttu-id="b25e7-139">Se você salvar suas alterações e reiniciar o aplicativo agora, após entrar, a interface do usuário será atualizada com o nome de exibição e o endereço de email do usuário.</span><span class="sxs-lookup"><span data-stu-id="b25e7-139">If you save your changes and restart the app now, after sign-in the UI is updated with the user's display name and email address.</span></span>
