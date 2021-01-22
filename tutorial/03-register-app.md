<!-- markdownlint-disable MD002 MD041 -->

<span data-ttu-id="edce2-101">Neste exercício, você criará um novo aplicativo nativo do Azure AD usando o centro de administração do Azure Active Directory.</span><span class="sxs-lookup"><span data-stu-id="edce2-101">In this exercise you will create a new Azure AD native application using the Azure Active Directory admin center.</span></span>

1. <span data-ttu-id="edce2-102">Abra um navegador, navegue até o [centro de administração do Azure Active Directory](https://aad.portal.azure.com) e faça logon usando uma **conta pessoal** (também conhecida como conta da Microsoft) ou **Conta Corporativa ou de Estudante**.</span><span class="sxs-lookup"><span data-stu-id="edce2-102">Open a browser and navigate to the [Azure Active Directory admin center](https://aad.portal.azure.com) and login using a **personal account** (aka: Microsoft Account) or **Work or School Account**.</span></span>

1. <span data-ttu-id="edce2-103">Selecione **Azure Active Directory** na navegação esquerda e selecione **Registros de aplicativos** em **Gerenciar**.</span><span class="sxs-lookup"><span data-stu-id="edce2-103">Select **Azure Active Directory** in the left-hand navigation, then select **App registrations** under **Manage**.</span></span>

    ![<span data-ttu-id="edce2-104">Uma captura de tela dos registros do aplicativo</span><span class="sxs-lookup"><span data-stu-id="edce2-104">A screenshot of the App registrations</span></span> ](images/aad-portal-app-registrations.png)

1. <span data-ttu-id="edce2-105">Selecione **Novo registro**.</span><span class="sxs-lookup"><span data-stu-id="edce2-105">Select **New registration**.</span></span> <span data-ttu-id="edce2-106">Na página **Registrar um aplicativo**, defina os valores da seguinte forma.</span><span class="sxs-lookup"><span data-stu-id="edce2-106">On the **Register an application** page, set the values as follows.</span></span>

    - <span data-ttu-id="edce2-107">Defina **Nome** para `iOS Swift Graph Tutorial`.</span><span class="sxs-lookup"><span data-stu-id="edce2-107">Set **Name** to `iOS Swift Graph Tutorial`.</span></span>
    - <span data-ttu-id="edce2-108">Defina **Tipos de conta com suporte** para **Contas em qualquer diretório organizacional e contas pessoais da Microsoft**.</span><span class="sxs-lookup"><span data-stu-id="edce2-108">Set **Supported account types** to **Accounts in any organizational directory and personal Microsoft accounts**.</span></span>
    - <span data-ttu-id="edce2-109">Deixe o **URI de Redirecionamento** vazio.</span><span class="sxs-lookup"><span data-stu-id="edce2-109">Leave **Redirect URI** empty.</span></span>

    ![Uma captura de tela da página Registrar um aplicativo](images/aad-register-an-app.png)

1. <span data-ttu-id="edce2-111">Selecione **Registrar**.</span><span class="sxs-lookup"><span data-stu-id="edce2-111">Select **Register**.</span></span> <span data-ttu-id="edce2-112">Na página do tutorial do **iOS Swift Graph,** copie o valor da ID do aplicativo **(cliente)** e salve-o, você precisará dele na próxima etapa.</span><span class="sxs-lookup"><span data-stu-id="edce2-112">On the **iOS Swift Graph Tutorial** page, copy the value of the **Application (client) ID** and save it, you will need it in the next step.</span></span>

    ![Uma captura de tela da ID do aplicativo do novo registro de aplicativo](images/aad-application-id.png)

1. <span data-ttu-id="edce2-114">Selecione **Autenticação** em **Gerenciar**.</span><span class="sxs-lookup"><span data-stu-id="edce2-114">Select **Authentication** under **Manage**.</span></span> <span data-ttu-id="edce2-115">Selecione **Adicionar uma plataforma** e, em **seguida, iOS/macOS.**</span><span class="sxs-lookup"><span data-stu-id="edce2-115">Select **Add a platform**, then **iOS / macOS**.</span></span>

1. <span data-ttu-id="edce2-116">Insira a ID do pacote do aplicativo e selecione **Configurar,** em seguida, selecione **Done**.</span><span class="sxs-lookup"><span data-stu-id="edce2-116">Enter your app's Bundle ID and select **Configure**, then select **Done**.</span></span>
