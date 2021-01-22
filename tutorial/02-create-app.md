<!-- markdownlint-disable MD002 MD041 -->

<span data-ttu-id="0c67f-101">Comece criando um novo projeto Swift.</span><span class="sxs-lookup"><span data-stu-id="0c67f-101">Begin by creating a new Swift project.</span></span>

1. <span data-ttu-id="0c67f-102">Abra o Xcode.</span><span class="sxs-lookup"><span data-stu-id="0c67f-102">Open Xcode.</span></span> <span data-ttu-id="0c67f-103">No menu **Arquivo,** selecione **Novo** e **Projeto.**</span><span class="sxs-lookup"><span data-stu-id="0c67f-103">On the **File** menu, select **New**, then **Project**.</span></span>
1. <span data-ttu-id="0c67f-104">Escolha o **modelo de** aplicativo e selecione **Próximo**.</span><span class="sxs-lookup"><span data-stu-id="0c67f-104">Choose the **App** template and select **Next**.</span></span>

    ![Uma captura de tela da caixa de diálogo de seleção do modelo Xcode](images/xcode-select-template.png)

1. <span data-ttu-id="0c67f-106">Definir o **nome do** produto e `GraphTutorial` o **idioma** como **Swift.**</span><span class="sxs-lookup"><span data-stu-id="0c67f-106">Set the **Product Name** to `GraphTutorial` and the **Language** to **Swift**.</span></span>
1. <span data-ttu-id="0c67f-107">Preencha os campos restantes e selecione **Next**.</span><span class="sxs-lookup"><span data-stu-id="0c67f-107">Fill in the remaining fields and select **Next**.</span></span>
1. <span data-ttu-id="0c67f-108">Escolha um local para o projeto e selecione **Criar**.</span><span class="sxs-lookup"><span data-stu-id="0c67f-108">Choose a location for the project and select **Create**.</span></span>

## <a name="install-dependencies"></a><span data-ttu-id="0c67f-109">Instalar dependências</span><span class="sxs-lookup"><span data-stu-id="0c67f-109">Install dependencies</span></span>

<span data-ttu-id="0c67f-110">Antes de continuar, instale algumas dependências adicionais que você usará mais tarde.</span><span class="sxs-lookup"><span data-stu-id="0c67f-110">Before moving on, install some additional dependencies that you will use later.</span></span>

- <span data-ttu-id="0c67f-111">[Biblioteca de Autenticação da Microsoft (MSAL) para iOS](https://github.com/AzureAD/microsoft-authentication-library-for-objc) para autenticação no Azure AD.</span><span class="sxs-lookup"><span data-stu-id="0c67f-111">[Microsoft Authentication Library (MSAL) for iOS](https://github.com/AzureAD/microsoft-authentication-library-for-objc) for authenticating to with Azure AD.</span></span>
- <span data-ttu-id="0c67f-112">[SDK do Microsoft Graph para Objetivo C](https://github.com/microsoftgraph/msgraph-sdk-objc) para fazer chamadas ao Microsoft Graph.</span><span class="sxs-lookup"><span data-stu-id="0c67f-112">[Microsoft Graph SDK for Objective C](https://github.com/microsoftgraph/msgraph-sdk-objc) for making calls to Microsoft Graph.</span></span>
- <span data-ttu-id="0c67f-113">[SDK de Modelos do Microsoft Graph para Objetivo C](https://github.com/microsoftgraph/msgraph-sdk-objc-models) para objetos fortemente digitados que representam recursos do Microsoft Graph, como usuários ou eventos.</span><span class="sxs-lookup"><span data-stu-id="0c67f-113">[Microsoft Graph Models SDK for Objective C](https://github.com/microsoftgraph/msgraph-sdk-objc-models) for strongly-typed objects representing Microsoft Graph resources like users or events.</span></span>

1. <span data-ttu-id="0c67f-114">Saia do Xcode.</span><span class="sxs-lookup"><span data-stu-id="0c67f-114">Quit Xcode.</span></span>
1. <span data-ttu-id="0c67f-115">Abra o Terminal e altere o diretório para o local do **seu projeto GraphTutorial.**</span><span class="sxs-lookup"><span data-stu-id="0c67f-115">Open Terminal and change the directory to the location of your **GraphTutorial** project.</span></span>
1. <span data-ttu-id="0c67f-116">Execute o seguinte comando para criar um Podfile.</span><span class="sxs-lookup"><span data-stu-id="0c67f-116">Run the following command to create a Podfile.</span></span>

    ```Shell
    pod init
    ```

1. <span data-ttu-id="0c67f-117">Abra o Podfile e adicione as linhas a seguir logo após a `use_frameworks!` linha.</span><span class="sxs-lookup"><span data-stu-id="0c67f-117">Open the Podfile and add the following lines just after the `use_frameworks!` line.</span></span>

    ```Ruby
    pod 'MSAL', '~> 1.1.13'
    pod 'MSGraphClientSDK', ' ~> 1.0.0'
    pod 'MSGraphClientModels', '~> 1.3.0'
    ```

1. <span data-ttu-id="0c67f-118">Salve o Podfile e execute o seguinte comando para instalar as dependências.</span><span class="sxs-lookup"><span data-stu-id="0c67f-118">Save the Podfile, then run the following command to install the dependencies.</span></span>

    ```Shell
    pod install
    ```

1. <span data-ttu-id="0c67f-119">Depois que o comando terminar, abra o **GraphTutorial.xcworkspace recém-criado** no Xcode.</span><span class="sxs-lookup"><span data-stu-id="0c67f-119">Once the command completes, open the newly created **GraphTutorial.xcworkspace** in Xcode.</span></span>

## <a name="design-the-app"></a><span data-ttu-id="0c67f-120">Projetar o aplicativo</span><span class="sxs-lookup"><span data-stu-id="0c67f-120">Design the app</span></span>

<span data-ttu-id="0c67f-121">Nesta seção, você criará as exibições para o aplicativo: uma página de login, um navegador da barra de guias, uma página de boas-vindas e uma página de calendário.</span><span class="sxs-lookup"><span data-stu-id="0c67f-121">In this section you will create the views for the app: a sign in page, a tab bar navigator, a welcome page, and a calendar page.</span></span> <span data-ttu-id="0c67f-122">Você também criará uma sobreposição do indicador de atividade.</span><span class="sxs-lookup"><span data-stu-id="0c67f-122">You'll also create an activity indicator overlay.</span></span>

### <a name="create-sign-in-page"></a><span data-ttu-id="0c67f-123">Criar página de login</span><span class="sxs-lookup"><span data-stu-id="0c67f-123">Create sign in page</span></span>

1. <span data-ttu-id="0c67f-124">Expanda **a pasta GraphTutorial** no Xcode e selecione **ViewController.swift.**</span><span class="sxs-lookup"><span data-stu-id="0c67f-124">Expand the **GraphTutorial** folder in Xcode, then select **ViewController.swift**.</span></span>
1. <span data-ttu-id="0c67f-125">No **Inspetor de Arquivos,** altere **o nome** do arquivo para `SignInViewController.swift` .</span><span class="sxs-lookup"><span data-stu-id="0c67f-125">In the **File Inspector**, change the **Name** of the file to `SignInViewController.swift`.</span></span>

    ![Uma captura de tela do Inspetor de Arquivos](images/rename-file.png)

1. <span data-ttu-id="0c67f-127">Abra **SignInViewController.swift e** substitua seu conteúdo pelo código a seguir.</span><span class="sxs-lookup"><span data-stu-id="0c67f-127">Open **SignInViewController.swift** and replace its contents with the following code.</span></span>

    ```Swift
    import UIKit

    class SignInViewController: UIViewController {

        override func viewDidLoad() {
            super.viewDidLoad()
            // Do any additional setup after loading the view.
        }

        @IBAction func signIn() {
            self.performSegue(withIdentifier: "userSignedIn", sender: nil)
        }
    }
    ```

1. <span data-ttu-id="0c67f-128">Abra **Main.storyboard**.</span><span class="sxs-lookup"><span data-stu-id="0c67f-128">Open **Main.storyboard**.</span></span> <span data-ttu-id="0c67f-129">Expanda **a cena do controlador** de exibição e selecione Controlador de **exibição.**</span><span class="sxs-lookup"><span data-stu-id="0c67f-129">Expand **View Controller Scene**, then select **View Controller**.</span></span>

    ![Captura de tela do Xcode com o Controlador de Exibição selecionado](images/storyboard-select-view-controller.png)

1. <span data-ttu-id="0c67f-131">Selecione o **Inspetor de Identidade,** em seguida, altere **o** menu suspenso de classe para **SignInViewController**.</span><span class="sxs-lookup"><span data-stu-id="0c67f-131">Select the **Identity Inspector**, then change the **Class** dropdown to **SignInViewController**.</span></span>

    ![Uma captura de tela do Inspetor de Identidade](images/change-class.png)

1. <span data-ttu-id="0c67f-133">Selecione a **biblioteca** e arraste um **botão para o** controlador de **exibição entrar.**</span><span class="sxs-lookup"><span data-stu-id="0c67f-133">Select the **Library**, then drag a **Button** onto the **Sign In View Controller**.</span></span>

    ![Uma captura de tela da Biblioteca no Xcode](images/add-button-to-view.png)

1. <span data-ttu-id="0c67f-135">Com o botão selecionado, selecione **o Inspetor de Atributos** e altere o **Título** do botão para `Sign In` .</span><span class="sxs-lookup"><span data-stu-id="0c67f-135">With the button selected, select the **Attributes Inspector** and change the **Title** of the button to `Sign In`.</span></span>

    ![Uma captura de tela do campo Título no Inspetor de Atributos no Xcode](images/set-button-title.png)

1. <span data-ttu-id="0c67f-137">Com o botão selecionado, selecione **o botão Alinhar** na parte inferior do storyboard.</span><span class="sxs-lookup"><span data-stu-id="0c67f-137">With the button selected, select the **Align** button at the bottom of the storyboard.</span></span> <span data-ttu-id="0c67f-138">Selecione **horizontalmente no contêiner** e **verticalmente** em restrições de contêiner, deixe seus valores como 0 e, em seguida, selecione **Adicionar 2 restrições**.</span><span class="sxs-lookup"><span data-stu-id="0c67f-138">Select both the **Horizontally in container** and **Vertically in container** constraints, leave their values as 0, then select **Add 2 constraints**.</span></span>

    ![Uma captura de tela das configurações de restrições de alinhamento no Xcode](images/add-alignment-constraints.png)

1. <span data-ttu-id="0c67f-140">Selecione o **Controlador de Exibição de Logo in,** em seguida, selecione o **Inspetor de Conexões.**</span><span class="sxs-lookup"><span data-stu-id="0c67f-140">Select the **Sign In View Controller**, then select the **Connections Inspector**.</span></span>
1. <span data-ttu-id="0c67f-141">Em **Ações Recebidas,** arraste o círculo não preenchido ao lado para **entrar** no botão.</span><span class="sxs-lookup"><span data-stu-id="0c67f-141">Under **Received Actions**, drag the unfilled circle next to **signIn** onto the button.</span></span> <span data-ttu-id="0c67f-142">Selecione **Tocar para Dentro** no menu pop-up.</span><span class="sxs-lookup"><span data-stu-id="0c67f-142">Select **Touch Up Inside** on the pop-up menu.</span></span>

    ![Uma captura de tela arrastando a ação de entrar para o botão no Xcode](images/connect-sign-in-button.png)

### <a name="create-tab-bar"></a><span data-ttu-id="0c67f-144">Criar barra de guias</span><span class="sxs-lookup"><span data-stu-id="0c67f-144">Create tab bar</span></span>

1. <span data-ttu-id="0c67f-145">Selecione a **Biblioteca** e arraste um Controlador de **Barra de Guias** para o storyboard.</span><span class="sxs-lookup"><span data-stu-id="0c67f-145">Select the **Library**, then drag a **Tab Bar Controller** onto the storyboard.</span></span>
1. <span data-ttu-id="0c67f-146">Selecione o **Controlador de Exibição de Logo in,** em seguida, selecione o **Inspetor de Conexões.**</span><span class="sxs-lookup"><span data-stu-id="0c67f-146">Select the **Sign In View Controller**, then select the **Connections Inspector**.</span></span>
1. <span data-ttu-id="0c67f-147">Em **Segues Disparados,** arraste o círculo não preenchido ao lado de **manual** para o Controlador da Barra **de Guias** no storyboard.</span><span class="sxs-lookup"><span data-stu-id="0c67f-147">Under **Triggered Segues**, drag the unfilled circle next to **manual** onto the **Tab Bar Controller** on the storyboard.</span></span> <span data-ttu-id="0c67f-148">Selecione **Apresentar Modalmente** no menu pop-up.</span><span class="sxs-lookup"><span data-stu-id="0c67f-148">Select **Present Modally** in the pop-up menu.</span></span>

    ![Uma captura de tela de arrastar um segue manual para o novo Controlador de Barra de Guias no Xcode](images/add-segue.png)

1. <span data-ttu-id="0c67f-150">Selecione o segue que você acabou de adicionar e selecione o **Inspetor de Atributos.**</span><span class="sxs-lookup"><span data-stu-id="0c67f-150">Select the segue you just added, then select the **Attributes Inspector**.</span></span> <span data-ttu-id="0c67f-151">Definir o **campo Identificador** como `userSignedIn` e definir **Apresentação** como **Tela Inteira.**</span><span class="sxs-lookup"><span data-stu-id="0c67f-151">Set the **Identifier** field to `userSignedIn`, and set **Presentation** to **Full Screen**.</span></span>

    ![Captura de tela do campo Identificador no Inspetor de Atributos no Xcode](images/set-segue-identifier.png)

1. <span data-ttu-id="0c67f-153">Selecione a **cena item 1** e, em seguida, selecione o **Inspetor de conexões**.</span><span class="sxs-lookup"><span data-stu-id="0c67f-153">Select the **Item 1 Scene**, then select the **Connections Inspector**.</span></span>
1. <span data-ttu-id="0c67f-154">Em **Segues Disparados,** arraste o círculo não preenchido ao lado de **manual** para o Controlador **de Modo** de Exibição entrar no storyboard.</span><span class="sxs-lookup"><span data-stu-id="0c67f-154">Under **Triggered Segues**, drag the unfilled circle next to **manual** onto the **Sign In View Controller** on the storyboard.</span></span> <span data-ttu-id="0c67f-155">Selecione **Apresentar Modalmente** no menu pop-up.</span><span class="sxs-lookup"><span data-stu-id="0c67f-155">Select **Present Modally** in the pop-up menu.</span></span>
1. <span data-ttu-id="0c67f-156">Selecione o segue que você acabou de adicionar e selecione o **Inspetor de Atributos.**</span><span class="sxs-lookup"><span data-stu-id="0c67f-156">Select the segue you just added, then select the **Attributes Inspector**.</span></span> <span data-ttu-id="0c67f-157">Definir o **campo Identificador** como `userSignedOut` e definir **Apresentação** como **Tela Inteira.**</span><span class="sxs-lookup"><span data-stu-id="0c67f-157">Set the **Identifier** field to `userSignedOut`, and set **Presentation** to **Full Screen**.</span></span>

### <a name="create-welcome-page"></a><span data-ttu-id="0c67f-158">Criar página de boas-vindas</span><span class="sxs-lookup"><span data-stu-id="0c67f-158">Create welcome page</span></span>

1. <span data-ttu-id="0c67f-159">Selecione o **arquivo Assets.xcassets.**</span><span class="sxs-lookup"><span data-stu-id="0c67f-159">Select the **Assets.xcassets** file.</span></span>
1. <span data-ttu-id="0c67f-160">No **menu Editor,** selecione **Adicionar Novo Ativo** e, em seguida, Conjunto de **Imagens.**</span><span class="sxs-lookup"><span data-stu-id="0c67f-160">On the **Editor** menu, select **Add New Asset**, then **Image Set**.</span></span>
1. <span data-ttu-id="0c67f-161">Selecione o novo **ativo de** imagem e use o Inspetor **de Atributos** para definir **seu nome** como `DefaultUserPhoto` .</span><span class="sxs-lookup"><span data-stu-id="0c67f-161">Select the new **Image** asset and use the **Attribute Inspector** to set its **Name** to `DefaultUserPhoto`.</span></span>
1. <span data-ttu-id="0c67f-162">Adicione qualquer imagem que você gosta de servir como uma foto de perfil de usuário padrão.</span><span class="sxs-lookup"><span data-stu-id="0c67f-162">Add any image you like to serve as a default user profile photo.</span></span>

    ![Uma captura de tela da exibição de ativo conjunto de imagens no Xcode](images/add-default-user-photo.png)

1. <span data-ttu-id="0c67f-164">Crie um novo **arquivo de classe Cocoa Touch** na pasta **GraphTutorial** chamada `WelcomeViewController` .</span><span class="sxs-lookup"><span data-stu-id="0c67f-164">Create a new **Cocoa Touch Class** file in the **GraphTutorial** folder named `WelcomeViewController`.</span></span> <span data-ttu-id="0c67f-165">Escolha **UIViewController** na **Subclasse do** campo.</span><span class="sxs-lookup"><span data-stu-id="0c67f-165">Choose **UIViewController** in the **Subclass of** field.</span></span>
1. <span data-ttu-id="0c67f-166">Abra **WelcomeViewController.swift** e substitua seu conteúdo pelo código a seguir.</span><span class="sxs-lookup"><span data-stu-id="0c67f-166">Open **WelcomeViewController.swift** and replace its contents with the following code.</span></span>

    ```Swift
    import UIKit

    class WelcomeViewController: UIViewController {

        @IBOutlet var userProfilePhoto: UIImageView!
        @IBOutlet var userDisplayName: UILabel!
        @IBOutlet var userEmail: UILabel!

        override func viewDidLoad() {
            super.viewDidLoad()

            // Do any additional setup after loading the view.

            // TEMPORARY
            self.userProfilePhoto.image = UIImage(imageLiteralResourceName: "DefaultUserPhoto")
            self.userDisplayName.text = "Default User"
            self.userEmail.text = "default@contoso.com"
        }

        @IBAction func signOut() {
            self.performSegue(withIdentifier: "userSignedOut", sender: nil)
        }
    }
    ```

1. <span data-ttu-id="0c67f-167">Abra **Main.storyboard**.</span><span class="sxs-lookup"><span data-stu-id="0c67f-167">Open **Main.storyboard**.</span></span> <span data-ttu-id="0c67f-168">Selecione a **Cena do Item 1** e, em seguida, selecione o **Inspetor de Identidade.**</span><span class="sxs-lookup"><span data-stu-id="0c67f-168">Select the **Item 1 Scene**, then select the **Identity Inspector**.</span></span> <span data-ttu-id="0c67f-169">Altere **o valor** de classe para **WelcomeViewController**.</span><span class="sxs-lookup"><span data-stu-id="0c67f-169">Change the **Class** value to **WelcomeViewController**.</span></span>
1. <span data-ttu-id="0c67f-170">Usando a **biblioteca**, adicione os seguintes itens à cena **item 1**.</span><span class="sxs-lookup"><span data-stu-id="0c67f-170">Using the **Library**, add the following items to the **Item 1 Scene**.</span></span>

    - <span data-ttu-id="0c67f-171">Um **exibição de imagem**</span><span class="sxs-lookup"><span data-stu-id="0c67f-171">One **Image View**</span></span>
    - <span data-ttu-id="0c67f-172">Dois **Rótulos**</span><span class="sxs-lookup"><span data-stu-id="0c67f-172">Two **Labels**</span></span>
    - <span data-ttu-id="0c67f-173">Um **botão**</span><span class="sxs-lookup"><span data-stu-id="0c67f-173">One **Button**</span></span>
1. <span data-ttu-id="0c67f-174">Usando o **Inspetor de Conexões,** faça as seguintes conexões.</span><span class="sxs-lookup"><span data-stu-id="0c67f-174">Using the **Connections Inspector**, make the following connections.</span></span>

    - <span data-ttu-id="0c67f-175">**Vincule a saída userDisplayName** ao primeiro rótulo.</span><span class="sxs-lookup"><span data-stu-id="0c67f-175">Link the **userDisplayName** outlet to the first label.</span></span>
    - <span data-ttu-id="0c67f-176">**Vincule a saída userEmail** ao segundo rótulo.</span><span class="sxs-lookup"><span data-stu-id="0c67f-176">Link the **userEmail** outlet to the second label.</span></span>
    - <span data-ttu-id="0c67f-177">Vincule **a saída userProfilePhoto** à exibição de imagem.</span><span class="sxs-lookup"><span data-stu-id="0c67f-177">Link the **userProfilePhoto** outlet to the image view.</span></span>
    - <span data-ttu-id="0c67f-178">**Vincule a ação** de logoff recebida ao botão Touch Up **Inside**.</span><span class="sxs-lookup"><span data-stu-id="0c67f-178">Link the **signOut** received action to the button's **Touch Up Inside**.</span></span>

1. <span data-ttu-id="0c67f-179">Selecione o visualizador de imagem e, em seguida, selecione o **Inspetor de tamanho.**</span><span class="sxs-lookup"><span data-stu-id="0c67f-179">Select the image view, then select the **Size Inspector**.</span></span>
1. <span data-ttu-id="0c67f-180">De definida **a largura** e **a altura** como 196.</span><span class="sxs-lookup"><span data-stu-id="0c67f-180">Set the **Width** and **Height** to 196.</span></span>
1. <span data-ttu-id="0c67f-181">Use o **botão Alinhar** para adicionar **a restrição Horizontalmente no** contêiner com um valor de 0.</span><span class="sxs-lookup"><span data-stu-id="0c67f-181">Use the **Align** button to add the **Horizontally in container** constraint with a value of 0.</span></span>
1. <span data-ttu-id="0c67f-182">Use o **botão Adicionar Novas Restrições** (ao lado do botão **Alinhar)** para adicionar as seguintes restrições:</span><span class="sxs-lookup"><span data-stu-id="0c67f-182">Use the **Add New Constraints** button (next to the **Align** button) to add the following constraints:</span></span>

    - <span data-ttu-id="0c67f-183">Alinhar de cima para: Área de Segurança, valor: 0</span><span class="sxs-lookup"><span data-stu-id="0c67f-183">Align Top to: Safe Area, value: 0</span></span>
    - <span data-ttu-id="0c67f-184">Espaço Inferior para: Nome de Exibição do Usuário, valor: Padrão</span><span class="sxs-lookup"><span data-stu-id="0c67f-184">Bottom Space to: User Display Name, value: Standard</span></span>
    - <span data-ttu-id="0c67f-185">Altura, valor: 196</span><span class="sxs-lookup"><span data-stu-id="0c67f-185">Height, value: 196</span></span>
    - <span data-ttu-id="0c67f-186">Largura, valor: 196</span><span class="sxs-lookup"><span data-stu-id="0c67f-186">Width, value: 196</span></span>

    ![Captura de tela das novas configurações de restrições no Xcode](./images/add-new-constraints.png)

1. <span data-ttu-id="0c67f-188">Selecione o primeiro rótulo e, em seguida, use o botão **Alinhar** para adicionar **a restrição Horizontalmente no** contêiner com um valor 0.</span><span class="sxs-lookup"><span data-stu-id="0c67f-188">Select the first label, then use the **Align** button to add the **Horizontally in container** constraint with a value of 0.</span></span>
1. <span data-ttu-id="0c67f-189">Use o **botão Adicionar Novas Restrições** para adicionar as seguintes restrições:</span><span class="sxs-lookup"><span data-stu-id="0c67f-189">Use the **Add New Constraints** button to add the following constraints:</span></span>

    - <span data-ttu-id="0c67f-190">Espaço Superior para: Foto do Perfil de Usuário, valor: Padrão</span><span class="sxs-lookup"><span data-stu-id="0c67f-190">Top Space to: User Profile Photo, value: Standard</span></span>
    - <span data-ttu-id="0c67f-191">Espaço Inferior para: Email do Usuário, valor: Padrão</span><span class="sxs-lookup"><span data-stu-id="0c67f-191">Bottom Space to: User Email, value: Standard</span></span>

1. <span data-ttu-id="0c67f-192">Selecione o segundo rótulo e selecione o **Inspetor de Atributos.**</span><span class="sxs-lookup"><span data-stu-id="0c67f-192">Select the second label, then select the **Attributes Inspector**.</span></span>
1. <span data-ttu-id="0c67f-193">Altere **a cor** para cinza escuro **e** altere a **fonte** para o **sistema 12.0**.</span><span class="sxs-lookup"><span data-stu-id="0c67f-193">Change the **Color** to **Dark Gray Color**, and change the **Font** to **System 12.0**.</span></span>
1. <span data-ttu-id="0c67f-194">Use o **botão Alinhar** para adicionar **a restrição Horizontalmente no** contêiner com um valor de 0.</span><span class="sxs-lookup"><span data-stu-id="0c67f-194">Use the **Align** button to add the **Horizontally in container** constraint with a value of 0.</span></span>
1. <span data-ttu-id="0c67f-195">Use o **botão Adicionar Novas Restrições** para adicionar as seguintes restrições:</span><span class="sxs-lookup"><span data-stu-id="0c67f-195">Use the **Add New Constraints** button to add the following constraints:</span></span>

    - <span data-ttu-id="0c67f-196">Espaço Superior para: Nome de Exibição do Usuário, valor: Padrão</span><span class="sxs-lookup"><span data-stu-id="0c67f-196">Top Space to: User Display Name, value: Standard</span></span>
    - <span data-ttu-id="0c67f-197">Espaço Inferior para: Sair, valor: 14</span><span class="sxs-lookup"><span data-stu-id="0c67f-197">Bottom Space to: Sign Out, value: 14</span></span>

1. <span data-ttu-id="0c67f-198">Selecione o botão e selecione o **Inspetor de Atributos.**</span><span class="sxs-lookup"><span data-stu-id="0c67f-198">Select the button, then select the **Attributes Inspector**.</span></span>
1. <span data-ttu-id="0c67f-199">Altere **o título** para `Sign Out` .</span><span class="sxs-lookup"><span data-stu-id="0c67f-199">Change the **Title** to `Sign Out`.</span></span>
1. <span data-ttu-id="0c67f-200">Use o **botão Alinhar** para adicionar **a restrição Horizontalmente no** contêiner com um valor de 0.</span><span class="sxs-lookup"><span data-stu-id="0c67f-200">Use the **Align** button to add the **Horizontally in container** constraint with a value of 0.</span></span>
1. <span data-ttu-id="0c67f-201">Use o **botão Adicionar Novas Restrições** para adicionar as seguintes restrições:</span><span class="sxs-lookup"><span data-stu-id="0c67f-201">Use the **Add New Constraints** button to add the following constraints:</span></span>

    - <span data-ttu-id="0c67f-202">Espaço Superior para: Email do Usuário, valor: 14</span><span class="sxs-lookup"><span data-stu-id="0c67f-202">Top Space to: User Email, value: 14</span></span>

1. <span data-ttu-id="0c67f-203">Selecione o item da barra de guias na parte inferior da cena e selecione o **Inspetor de Atributos.**</span><span class="sxs-lookup"><span data-stu-id="0c67f-203">Select the tab bar item at the bottom of the scene, then select the **Attributes Inspector**.</span></span> <span data-ttu-id="0c67f-204">Altere **o título** para `Me` .</span><span class="sxs-lookup"><span data-stu-id="0c67f-204">Change the **Title** to `Me`.</span></span>

<span data-ttu-id="0c67f-205">A cena de boas-vindas deve ser semelhante a esta quando você terminar.</span><span class="sxs-lookup"><span data-stu-id="0c67f-205">The welcome scene should look similar to this once you're done.</span></span>

![Uma captura de tela do layout da cena de boas-vindas](images/welcome-scene-layout.png)

### <a name="create-calendar-page"></a><span data-ttu-id="0c67f-207">Criar página de calendário</span><span class="sxs-lookup"><span data-stu-id="0c67f-207">Create calendar page</span></span>

1. <span data-ttu-id="0c67f-208">Crie um novo **arquivo de classe Cocoa Touch** na pasta **GraphTutorial** chamada `CalendarViewController` .</span><span class="sxs-lookup"><span data-stu-id="0c67f-208">Create a new **Cocoa Touch Class** file in the **GraphTutorial** folder named `CalendarViewController`.</span></span> <span data-ttu-id="0c67f-209">Escolha **UIViewController** na **Subclasse do** campo.</span><span class="sxs-lookup"><span data-stu-id="0c67f-209">Choose **UIViewController** in the **Subclass of** field.</span></span>
1. <span data-ttu-id="0c67f-210">Abra **CalendarViewController.swift** e substitua seu conteúdo pelo código a seguir.</span><span class="sxs-lookup"><span data-stu-id="0c67f-210">Open **CalendarViewController.swift** and replace its contents with the following code.</span></span>

    ```Swift
    import UIKit

    class CalendarViewController: UIViewController {

        @IBOutlet var calendarJSON: UITextView!

        override func viewDidLoad() {
            super.viewDidLoad()

            // Do any additional setup after loading the view.

            // TEMPORARY
            calendarJSON.text = "Calendar"
            calendarJSON.sizeToFit()
        }
    }
    ```

1. <span data-ttu-id="0c67f-211">Abra **Main.storyboard**.</span><span class="sxs-lookup"><span data-stu-id="0c67f-211">Open **Main.storyboard**.</span></span> <span data-ttu-id="0c67f-212">Selecione a **Cena do Item 2** e, em seguida, selecione o **Inspetor de Identidade.**</span><span class="sxs-lookup"><span data-stu-id="0c67f-212">Select the **Item 2 Scene**, then select the **Identity Inspector**.</span></span> <span data-ttu-id="0c67f-213">Altere **o valor** de classe para **CalendarViewController**.</span><span class="sxs-lookup"><span data-stu-id="0c67f-213">Change the **Class** value to **CalendarViewController**.</span></span>
1. <span data-ttu-id="0c67f-214">Usando a **biblioteca**, adicione um **exibição de texto** à cena item **2**.</span><span class="sxs-lookup"><span data-stu-id="0c67f-214">Using the **Library**, add a **Text View** to the **Item 2 Scene**.</span></span>
1. <span data-ttu-id="0c67f-215">Selecione o exibição de texto que você acabou de adicionar.</span><span class="sxs-lookup"><span data-stu-id="0c67f-215">Select the text view you just added.</span></span> <span data-ttu-id="0c67f-216">No **menu Editor,** escolha **Inserir em e,** em seguida, Exibir **Rolagem.**</span><span class="sxs-lookup"><span data-stu-id="0c67f-216">On the **Editor** menu, choose **Embed In**, then **Scroll View**.</span></span>
1. <span data-ttu-id="0c67f-217">Reize a exibição de rolagem e a exibição de texto para assumir toda a tela.</span><span class="sxs-lookup"><span data-stu-id="0c67f-217">Resize the scroll view and text view to take up the entire screen.</span></span>
1. <span data-ttu-id="0c67f-218">Usando o **Inspetor de Conexões,** conecte **a saída calendarJSON** ao exibição de texto.</span><span class="sxs-lookup"><span data-stu-id="0c67f-218">Using the **Connections Inspector**, connect the **calendarJSON** outlet to the text view.</span></span>
1. <span data-ttu-id="0c67f-219">Selecione o item da barra de guias na parte inferior da cena e selecione o **Inspetor de Atributos.**</span><span class="sxs-lookup"><span data-stu-id="0c67f-219">Select the tab bar item at the bottom of the scene, then select the **Attributes Inspector**.</span></span> <span data-ttu-id="0c67f-220">Altere **o título** para `Calendar` .</span><span class="sxs-lookup"><span data-stu-id="0c67f-220">Change the **Title** to `Calendar`.</span></span>
1. <span data-ttu-id="0c67f-221">No **menu Editor,** selecione Resolver Problemas **de Layout** Automático e, em seguida, selecione Adicionar Restrições **Ausentes** abaixo de Todos os Exibições no Controlador de Modo **de Exibição de Calendário.**</span><span class="sxs-lookup"><span data-stu-id="0c67f-221">On the **Editor** menu, select **Resolve Auto Layout Issues**, then select **Add Missing Constraints** underneath **All Views in Calendar View Controller**.</span></span>

<span data-ttu-id="0c67f-222">A cena do calendário deve ser semelhante a esta quando você terminar.</span><span class="sxs-lookup"><span data-stu-id="0c67f-222">The calendar scene should look similar to this once you're done.</span></span>

![Uma captura de tela do layout da cena calendário](images/calendar-scene-layout.png)

### <a name="create-activity-indicator"></a><span data-ttu-id="0c67f-224">Criar indicador de atividade</span><span class="sxs-lookup"><span data-stu-id="0c67f-224">Create activity indicator</span></span>

1. <span data-ttu-id="0c67f-225">Crie um novo **arquivo de classe Cocoa Touch** na pasta **GraphTutorial** chamada `SpinnerViewController` .</span><span class="sxs-lookup"><span data-stu-id="0c67f-225">Create a new **Cocoa Touch Class** file in the **GraphTutorial** folder named `SpinnerViewController`.</span></span> <span data-ttu-id="0c67f-226">Escolha **UIViewController** na **Subclasse do** campo.</span><span class="sxs-lookup"><span data-stu-id="0c67f-226">Choose **UIViewController** in the **Subclass of** field.</span></span>
1. <span data-ttu-id="0c67f-227">Abra **SpinnerViewController.swift** e substitua seu conteúdo pelo código a seguir.</span><span class="sxs-lookup"><span data-stu-id="0c67f-227">Open **SpinnerViewController.swift** and replace its contents with the following code.</span></span>

    :::code language="swift" source="../demo/GraphTutorial/GraphTutorial/SpinnerViewController.swift" id="SpinnerSnippet":::

## <a name="test-the-app"></a><span data-ttu-id="0c67f-228">O aplicativo de teste</span><span class="sxs-lookup"><span data-stu-id="0c67f-228">Test the app</span></span>

<span data-ttu-id="0c67f-229">Salve suas alterações e iniciar o aplicativo.</span><span class="sxs-lookup"><span data-stu-id="0c67f-229">Save your changes and launch the app.</span></span> <span data-ttu-id="0c67f-230">Você deve ser capaz de se  mover entre as telas usando os botões Entrar e **Sair** e a barra de guias.</span><span class="sxs-lookup"><span data-stu-id="0c67f-230">You should be able to move between the screens using the **Sign In** and **Sign Out** buttons and the tab bar.</span></span>

![Capturas de tela do aplicativo](images/app-screens.png)
