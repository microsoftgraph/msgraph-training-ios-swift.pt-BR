<!-- markdownlint-disable MD002 MD041 -->

<span data-ttu-id="7c69c-101">Neste exercício, você incorporará o Microsoft Graph ao aplicativo.</span><span class="sxs-lookup"><span data-stu-id="7c69c-101">In this exercise you will incorporate the Microsoft Graph into the application.</span></span> <span data-ttu-id="7c69c-102">Para este aplicativo, você usará o [SDK](https://github.com/microsoftgraph/msgraph-sdk-objc) do Microsoft Graph para o Objetivo C para fazer chamadas ao Microsoft Graph.</span><span class="sxs-lookup"><span data-stu-id="7c69c-102">For this application, you will use the [Microsoft Graph SDK for Objective C](https://github.com/microsoftgraph/msgraph-sdk-objc) to make calls to Microsoft Graph.</span></span>

## <a name="get-calendar-events-from-outlook"></a><span data-ttu-id="7c69c-103">Obter eventos de calendário do Outlook</span><span class="sxs-lookup"><span data-stu-id="7c69c-103">Get calendar events from Outlook</span></span>

<span data-ttu-id="7c69c-104">Nesta seção, você estenderá a classe para adicionar uma função para obter os eventos do usuário para a semana atual e atualizar para `GraphManager` `CalendarViewController` usar essas novas funções.</span><span class="sxs-lookup"><span data-stu-id="7c69c-104">In this section you will extend the `GraphManager` class to add a function to get the user's events for the current week and update `CalendarViewController` to use these new functions.</span></span>

1. <span data-ttu-id="7c69c-105">Abra **GraphManager.swift** e adicione o método a seguir à `GraphManager` classe.</span><span class="sxs-lookup"><span data-stu-id="7c69c-105">Open **GraphManager.swift** and add the following method to the `GraphManager` class.</span></span>

    ```Swift
    public func getCalendarView(viewStart: String,
                                viewEnd: String,
                                completion: @escaping(Data?, Error?) -> Void) {
        // GET /me/calendarview
        // Set start and end of the view
        let start = "startDateTime=\(viewStart)"
        let end = "endDateTime=\(viewEnd)"
        // Only return these fields in results
        let select = "$select=subject,organizer,start,end"
        // Sort results by when they were created, newest first
        let orderBy = "$orderby=start/dateTime"
        // Request at most 25 results
        let top = "$top=25"

        let eventsRequest = NSMutableURLRequest(url: URL(string: "\(MSGraphBaseURL)/me/calendarview?\(start)&\(end)&\(select)&\(orderBy)&\(top)")!)

        // Add the Prefer: outlook.timezone header to get start and end times
        // in user's time zone
        eventsRequest.addValue("outlook.timezone=\"\(self.userTimeZone)\"", forHTTPHeaderField: "Prefer")

        let eventsDataTask = MSURLSessionDataTask(request: eventsRequest, client: self.client, completion: {
            (data: Data?, response: URLResponse?, graphError: Error?) in
            guard let eventsData = data, graphError == nil else {
                completion(nil, graphError)
                return
            }

            // TEMPORARY
            completion(eventsData, nil)
        })

        // Execute the request
        eventsDataTask?.execute()
    }
    ```

    > [!NOTE]
    > <span data-ttu-id="7c69c-106">Considere o que o código `getCalendarView` está fazendo.</span><span class="sxs-lookup"><span data-stu-id="7c69c-106">Consider what the code in `getCalendarView` is doing.</span></span>
    >
    > - <span data-ttu-id="7c69c-107">A URL que será chamada é `/v1.0/me/calendarview` .</span><span class="sxs-lookup"><span data-stu-id="7c69c-107">The URL that will be called is `/v1.0/me/calendarview`.</span></span>
    >   - <span data-ttu-id="7c69c-108">Os `startDateTime` `endDateTime` parâmetros e consulta definem o início e o fim da exibição de calendário.</span><span class="sxs-lookup"><span data-stu-id="7c69c-108">The `startDateTime` and `endDateTime` query parameters define the start and end of the calendar view.</span></span>
    >   - <span data-ttu-id="7c69c-109">O `select` parâmetro de consulta limita os campos retornados para cada evento a apenas aqueles que o ponto de exibição realmente usará.</span><span class="sxs-lookup"><span data-stu-id="7c69c-109">The `select` query parameter limits the fields returned for each events to just those the view will actually use.</span></span>
    >   - <span data-ttu-id="7c69c-110">O `orderby` parâmetro de consulta classifica os resultados por hora de início.</span><span class="sxs-lookup"><span data-stu-id="7c69c-110">The `orderby` query parameter sorts the results by start time.</span></span>
    >   - <span data-ttu-id="7c69c-111">O `top` parâmetro de consulta solicita 25 resultados por página.</span><span class="sxs-lookup"><span data-stu-id="7c69c-111">The `top` query parameter requests 25 results per page.</span></span>
    >   - <span data-ttu-id="7c69c-112">o header faz com que o Microsoft Graph retorne as horas de início e término de cada evento `Prefer: outlook.timezone` no fuso horário do usuário.</span><span class="sxs-lookup"><span data-stu-id="7c69c-112">the `Prefer: outlook.timezone` header causes the Microsoft Graph to return the start and end times of each event in the user's time zone.</span></span>

1. <span data-ttu-id="7c69c-113">Crie um novo **Arquivo Swift no** projeto **GraphTutorial** chamado **GraphToIana.swift.**</span><span class="sxs-lookup"><span data-stu-id="7c69c-113">Create a new **Swift File** in the **GraphTutorial** project named **GraphToIana.swift**.</span></span> <span data-ttu-id="7c69c-114">Adicione o código a seguir a esse arquivo.</span><span class="sxs-lookup"><span data-stu-id="7c69c-114">Add the following code to the file.</span></span>

    :::code language="swift" source="../demo/GraphTutorial/GraphTutorial/GraphToIana.swift" id="GraphToIanaSnippet":::

    <span data-ttu-id="7c69c-115">Isso faz uma pesquisa simples para encontrar um identificador de fuso horário IANA com base no nome do fuso horário retornado pelo Microsoft Graph.</span><span class="sxs-lookup"><span data-stu-id="7c69c-115">This does a simple lookup to find an IANA time zone identifier based on the time zone name returned by Microsoft Graph.</span></span>

1. <span data-ttu-id="7c69c-116">Abra **CalendarViewController.swift** e substitua seu conteúdo inteiro pelo código a seguir.</span><span class="sxs-lookup"><span data-stu-id="7c69c-116">Open **CalendarViewController.swift** and replace its entire contents with the following code.</span></span>

    ```Swift
    import UIKit
    import MSGraphClientModels

    class CalendarViewController: UIViewController {

        @IBOutlet var calendarJSON: UITextView!

        private let spinner = SpinnerViewController()

        override func viewDidLoad() {
            super.viewDidLoad()

            // Do any additional setup after loading the view.
            self.spinner.start(container: self)

            // Calculate the start and end of the current week
            let timeZone = GraphToIana.getIanaIdentifier(graphIdentifer: GraphManager.instance.userTimeZone)
            let now = Date()
            var calendar = Calendar(identifier: .gregorian)
            calendar.timeZone = TimeZone(identifier: timeZone)!

            let startOfWeek = calendar.dateComponents([.calendar, .yearForWeekOfYear, .weekOfYear], from: now).date!
            let endOfWeek = calendar.date(byAdding: .day, value: 7, to: startOfWeek)!

            // Convert start and end to ISO 8601 strings
            let isoFormatter = ISO8601DateFormatter()
            let viewStart = isoFormatter.string(from: startOfWeek)
            let viewEnd = isoFormatter.string(from: endOfWeek)

            GraphManager.instance.getCalendarView(viewStart: viewStart, viewEnd: viewEnd) {
                (data: Data?, error: Error?) in
                DispatchQueue.main.async {
                    self.spinner.stop()

                    // TEMPORARY
                    guard let eventsData = data, error == nil else {
                        self.calendarJSON.text = error.debugDescription
                        return
                    }

                    let jsonString = String(data: eventsData, encoding: .utf8)
                    self.calendarJSON.text = jsonString
                    self.calendarJSON.sizeToFit()
                }
            }
        }
    }
    ```

<span data-ttu-id="7c69c-117">Agora você pode executar o aplicativo, entrar e tocar no item **de** navegação calendário no menu.</span><span class="sxs-lookup"><span data-stu-id="7c69c-117">You can now run the app, sign in, and tap the **Calendar** navigation item in the menu.</span></span> <span data-ttu-id="7c69c-118">Você deverá ver um despejo JSON dos eventos no aplicativo.</span><span class="sxs-lookup"><span data-stu-id="7c69c-118">You should see a JSON dump of the events in the app.</span></span>

## <a name="display-the-results"></a><span data-ttu-id="7c69c-119">Exibir os resultados</span><span class="sxs-lookup"><span data-stu-id="7c69c-119">Display the results</span></span>

<span data-ttu-id="7c69c-120">Agora você pode substituir o despejo JSON por algo para exibir os resultados de maneira amigável.</span><span class="sxs-lookup"><span data-stu-id="7c69c-120">Now you can replace the JSON dump with something to display the results in a user-friendly manner.</span></span> <span data-ttu-id="7c69c-121">Nesta seção, você modificará a função para retornar objetos fortemente digitados e modificará para usar um exibição de tabela para `getCalendarView` `CalendarViewController` renderizar os eventos.</span><span class="sxs-lookup"><span data-stu-id="7c69c-121">In this section, you will modify the `getCalendarView` function to return strongly-typed objects, and modify `CalendarViewController` to use a table view to render the events.</span></span>

1. <span data-ttu-id="7c69c-122">Abra **GraphManager.swift.**</span><span class="sxs-lookup"><span data-stu-id="7c69c-122">Open **GraphManager.swift**.</span></span> <span data-ttu-id="7c69c-123">Substitua a função `getCalendarView` existente pelo seguinte.</span><span class="sxs-lookup"><span data-stu-id="7c69c-123">Replace the existing `getCalendarView` function with the following.</span></span>

    :::code language="swift" source="../demo/GraphTutorial/GraphTutorial/GraphManager.swift" id="GetEventsSnippet" highlight="3,28-49":::

1. <span data-ttu-id="7c69c-124">Crie um novo **arquivo de classe Cocoa Touch** no projeto **GraphTutorial** chamado `CalendarTableViewController.swift` .</span><span class="sxs-lookup"><span data-stu-id="7c69c-124">Create a new **Cocoa Touch Class** file in the **GraphTutorial** project named `CalendarTableViewController.swift`.</span></span> <span data-ttu-id="7c69c-125">Escolha **UITableViewController** na **Subclasse do** campo.</span><span class="sxs-lookup"><span data-stu-id="7c69c-125">Choose **UITableViewController** in the **Subclass of** field.</span></span>

1. <span data-ttu-id="7c69c-126">Abra **CalendarTableViewController.swift** e substitua seu conteúdo pelo seguinte.</span><span class="sxs-lookup"><span data-stu-id="7c69c-126">Open **CalendarTableViewController.swift** and replace its contents with the following.</span></span>

    :::code language="swift" source="../demo/GraphTutorial/GraphTutorial/CalendarTableViewController.swift" id="CalendarTableViewControllerSnippet":::

1. <span data-ttu-id="7c69c-127">Crie um novo **arquivo de classe Cocoa Touch** no projeto **GraphTutorial** chamado `CalendarTableViewCell.swift` .</span><span class="sxs-lookup"><span data-stu-id="7c69c-127">Create a new **Cocoa Touch Class** file in the **GraphTutorial** project named `CalendarTableViewCell.swift`.</span></span> <span data-ttu-id="7c69c-128">Escolha **UITableViewCell** na **Subclasse do** campo.</span><span class="sxs-lookup"><span data-stu-id="7c69c-128">Choose **UITableViewCell** in the **Subclass of** field.</span></span>

1. <span data-ttu-id="7c69c-129">Abra **CalendarTableViewCell.swift** e adicione as seguintes propriedades à `CalendarTableViewCell` classe.</span><span class="sxs-lookup"><span data-stu-id="7c69c-129">Open **CalendarTableViewCell.swift** and add the following properties to the `CalendarTableViewCell` class.</span></span>

    :::code language="swift" source="../demo/GraphTutorial/GraphTutorial/CalendarTableViewCell.swift" id="PropertiesSnippet":::

1. <span data-ttu-id="7c69c-130">Abra **Main.storyboard** e localize a **cena do calendário.**</span><span class="sxs-lookup"><span data-stu-id="7c69c-130">Open **Main.storyboard** and locate the **Calendar Scene**.</span></span> <span data-ttu-id="7c69c-131">Exclua o visualizador de rolagem do visualização raiz.</span><span class="sxs-lookup"><span data-stu-id="7c69c-131">Delete the scroll view from the root view.</span></span>
1. <span data-ttu-id="7c69c-132">Usando a **Biblioteca,** adicione uma **Barra de Navegação** à parte superior do modo de exibição.</span><span class="sxs-lookup"><span data-stu-id="7c69c-132">Using the **Library**, add a **Navigation Bar** to the top of the view.</span></span>
1. <span data-ttu-id="7c69c-133">Clique duas vezes no **título** na barra de navegação e atualize-o para `Calendar` .</span><span class="sxs-lookup"><span data-stu-id="7c69c-133">Double-click the **Title** in the navigation bar and update it to `Calendar`.</span></span>
1. <span data-ttu-id="7c69c-134">Usando a **biblioteca**, adicione um **item de** botão de barra ao lado direito da barra de navegação.</span><span class="sxs-lookup"><span data-stu-id="7c69c-134">Using the **Library**, add a **Bar Button Item** to the right-hand side of the navigation bar.</span></span>
1. <span data-ttu-id="7c69c-135">Selecione o novo botão de barra e selecione o **Inspetor de Atributos.**</span><span class="sxs-lookup"><span data-stu-id="7c69c-135">Select the new bar button, then select the **Attributes Inspector**.</span></span> <span data-ttu-id="7c69c-136">Alterar **Imagem para** **mais**.</span><span class="sxs-lookup"><span data-stu-id="7c69c-136">Change **Image** to **plus**.</span></span>
1. <span data-ttu-id="7c69c-137">Adicione um **modo de exibição** de contêiner **da Biblioteca** ao modo de exibição na barra de navegação.</span><span class="sxs-lookup"><span data-stu-id="7c69c-137">Add a **Container View** from the **Library** to the view under the navigation bar.</span></span> <span data-ttu-id="7c69c-138">Reize a exibição do contêiner para que pegue todo o espaço restante no ponto de vista.</span><span class="sxs-lookup"><span data-stu-id="7c69c-138">Resize the container view to take all of the remaining space in the view.</span></span>
1. <span data-ttu-id="7c69c-139">Definir restrições na barra de navegação e no modo de exibição de contêiner da seguinte forma.</span><span class="sxs-lookup"><span data-stu-id="7c69c-139">Set constraints on the navigation bar and container view as follows.</span></span>

    - <span data-ttu-id="7c69c-140">**Barra de Navegação**</span><span class="sxs-lookup"><span data-stu-id="7c69c-140">**Navigation Bar**</span></span>
        - <span data-ttu-id="7c69c-141">Adicionar restrição: Altura, valor: 44</span><span class="sxs-lookup"><span data-stu-id="7c69c-141">Add constraint: Height, value: 44</span></span>
        - <span data-ttu-id="7c69c-142">Restrição Adicionar: Espaço à frente da Área de Segurança, valor: 0</span><span class="sxs-lookup"><span data-stu-id="7c69c-142">Add constraint: Leading space to Safe Area, value: 0</span></span>
        - <span data-ttu-id="7c69c-143">Adicionar restrição: Espaço à trailing para a Área de Segurança, valor: 0</span><span class="sxs-lookup"><span data-stu-id="7c69c-143">Add constraint: Trailing space to Safe Area, value: 0</span></span>
        - <span data-ttu-id="7c69c-144">Restrição Adicionar: Espaço superior à Área de Segurança, valor: 0</span><span class="sxs-lookup"><span data-stu-id="7c69c-144">Add constraint: Top space to Safe Area, value: 0</span></span>
    - <span data-ttu-id="7c69c-145">**Exibição de Contêiner**</span><span class="sxs-lookup"><span data-stu-id="7c69c-145">**Container View**</span></span>
        - <span data-ttu-id="7c69c-146">Restrição Adicionar: Espaço à frente da Área de Segurança, valor: 0</span><span class="sxs-lookup"><span data-stu-id="7c69c-146">Add constraint: Leading space to Safe Area, value: 0</span></span>
        - <span data-ttu-id="7c69c-147">Adicionar restrição: Espaço à trailing para a Área de Segurança, valor: 0</span><span class="sxs-lookup"><span data-stu-id="7c69c-147">Add constraint: Trailing space to Safe Area, value: 0</span></span>
        - <span data-ttu-id="7c69c-148">Restrição Adicionar: Espaço superior à Barra de Navegação Inferior, valor: 0</span><span class="sxs-lookup"><span data-stu-id="7c69c-148">Add constraint: Top space to Navigation Bar Bottom, value: 0</span></span>
        - <span data-ttu-id="7c69c-149">Adicionar restrição: Espaço inferior à Área de Segurança, valor: 0</span><span class="sxs-lookup"><span data-stu-id="7c69c-149">Add constraint: Bottom space to Safe Area, value: 0</span></span>

1. <span data-ttu-id="7c69c-150">Localize o segundo controlador de exibição adicionado ao storyboard quando você adicionou o ponto de vista do contêiner.</span><span class="sxs-lookup"><span data-stu-id="7c69c-150">Locate the second view controller added to the storyboard when you added the container view.</span></span> <span data-ttu-id="7c69c-151">Ele é conectado à **cena do calendário** por um segue de insusuidade.</span><span class="sxs-lookup"><span data-stu-id="7c69c-151">It is connected to the **Calendar Scene** by an embed segue.</span></span> <span data-ttu-id="7c69c-152">Selecione este controlador e use o **Inspetor de Identidade** para alterar a **classe** para **CalendarTableViewController**.</span><span class="sxs-lookup"><span data-stu-id="7c69c-152">Select this controller and use the **Identity Inspector** to change **Class** to **CalendarTableViewController**.</span></span>
1. <span data-ttu-id="7c69c-153">Exclua **o view** do controlador **de exibição de tabela de calendário.**</span><span class="sxs-lookup"><span data-stu-id="7c69c-153">Delete the **View** from the **Calendar Table View Controller**.</span></span>
1. <span data-ttu-id="7c69c-154">Adicione um **exibição de tabela** da **biblioteca ao** controlador **de exibição de tabela de calendário.**</span><span class="sxs-lookup"><span data-stu-id="7c69c-154">Add a **Table View** from the **Library** to the **Calendar Table View Controller**.</span></span>
1. <span data-ttu-id="7c69c-155">Selecione o exibição de tabela e selecione o **Inspetor de Atributos.**</span><span class="sxs-lookup"><span data-stu-id="7c69c-155">Select the table view, then select the **Attributes Inspector**.</span></span> <span data-ttu-id="7c69c-156">Definir **células de protótipo** como **1**.</span><span class="sxs-lookup"><span data-stu-id="7c69c-156">Set **Prototype Cells** to **1**.</span></span>
1. <span data-ttu-id="7c69c-157">Arraste a borda inferior da célula de protótipo para lhe dar uma área maior para trabalhar.</span><span class="sxs-lookup"><span data-stu-id="7c69c-157">Drag the bottom edge of the prototype cell to give you a larger area to work with.</span></span>
1. <span data-ttu-id="7c69c-158">Use a **Biblioteca para** adicionar três **Rótulos** à célula de protótipo.</span><span class="sxs-lookup"><span data-stu-id="7c69c-158">Use the **Library** to add three **Labels** to the prototype cell.</span></span>
1. <span data-ttu-id="7c69c-159">Selecione a célula de protótipo e, em seguida, selecione o **Inspetor de Identidade.**</span><span class="sxs-lookup"><span data-stu-id="7c69c-159">Select the prototype cell, then select the **Identity Inspector**.</span></span> <span data-ttu-id="7c69c-160">Alterar **classe** para **CalendarTableViewCell**.</span><span class="sxs-lookup"><span data-stu-id="7c69c-160">Change **Class** to **CalendarTableViewCell**.</span></span>
1. <span data-ttu-id="7c69c-161">Selecione o **Inspetor de Atributos** e de definir o **identificador** como `EventCell` .</span><span class="sxs-lookup"><span data-stu-id="7c69c-161">Select the **Attributes Inspector** and set **Identifier** to `EventCell`.</span></span>
1. <span data-ttu-id="7c69c-162">Com o **EventCell** selecionado, selecione o Inspetor de Conexões e conecte-se, e aos rótulos que você adicionou à célula no  `durationLabel` `organizerLabel` `subjectLabel` storyboard.</span><span class="sxs-lookup"><span data-stu-id="7c69c-162">With the **EventCell** selected, select the **Connections Inspector** and connect `durationLabel`, `organizerLabel`, and `subjectLabel` to the labels you added to the cell on the storyboard.</span></span>
1. <span data-ttu-id="7c69c-163">De definidas as propriedades e restrições nos três rótulos da seguinte forma.</span><span class="sxs-lookup"><span data-stu-id="7c69c-163">Set the properties and constraints on the three labels as follows.</span></span>

    - <span data-ttu-id="7c69c-164">**Rótulo de Assunto**</span><span class="sxs-lookup"><span data-stu-id="7c69c-164">**Subject Label**</span></span>
        - <span data-ttu-id="7c69c-165">Adicionar restrição: espaço à esquerda do visualização de conteúdo à margem esquerda, valor: 0</span><span class="sxs-lookup"><span data-stu-id="7c69c-165">Add constraint: Leading space to Content View Leading Margin, value: 0</span></span>
        - <span data-ttu-id="7c69c-166">Adicionar restrição: espaço à direita para a margem à direita do visualização de conteúdo, valor: 0</span><span class="sxs-lookup"><span data-stu-id="7c69c-166">Add constraint: Trailing space to Content View Trailing Margin, value: 0</span></span>
        - <span data-ttu-id="7c69c-167">Adicionar restrição: Espaço superior à Margem Superior do Exibição de Conteúdo, valor: 0</span><span class="sxs-lookup"><span data-stu-id="7c69c-167">Add constraint: Top space to Content View Top Margin, value: 0</span></span>
    - <span data-ttu-id="7c69c-168">**Rótulo do Organizador**</span><span class="sxs-lookup"><span data-stu-id="7c69c-168">**Organizer Label**</span></span>
        - <span data-ttu-id="7c69c-169">Fonte: Sistema 12.0</span><span class="sxs-lookup"><span data-stu-id="7c69c-169">Font: System 12.0</span></span>
        - <span data-ttu-id="7c69c-170">Restrição Adicionar: Altura, valor: 15</span><span class="sxs-lookup"><span data-stu-id="7c69c-170">Add constraint: Height, value: 15</span></span>
        - <span data-ttu-id="7c69c-171">Adicionar restrição: espaço à esquerda do visualização de conteúdo à margem esquerda, valor: 0</span><span class="sxs-lookup"><span data-stu-id="7c69c-171">Add constraint: Leading space to Content View Leading Margin, value: 0</span></span>
        - <span data-ttu-id="7c69c-172">Adicionar restrição: espaço à direita para a margem à direita do visualização de conteúdo, valor: 0</span><span class="sxs-lookup"><span data-stu-id="7c69c-172">Add constraint: Trailing space to Content View Trailing Margin, value: 0</span></span>
        - <span data-ttu-id="7c69c-173">Adicionar restrição: Espaço superior para o Rótulo de Assunto Inferior, valor: Padrão</span><span class="sxs-lookup"><span data-stu-id="7c69c-173">Add constraint: Top space to Subject Label Bottom, value: Standard</span></span>
    - <span data-ttu-id="7c69c-174">**Rótulo de Duração**</span><span class="sxs-lookup"><span data-stu-id="7c69c-174">**Duration Label**</span></span>
        - <span data-ttu-id="7c69c-175">Fonte: Sistema 12.0</span><span class="sxs-lookup"><span data-stu-id="7c69c-175">Font: System 12.0</span></span>
        - <span data-ttu-id="7c69c-176">Cor: Cor Cinza Escuro</span><span class="sxs-lookup"><span data-stu-id="7c69c-176">Color: Dark Gray Color</span></span>
        - <span data-ttu-id="7c69c-177">Restrição Adicionar: Altura, valor: 15</span><span class="sxs-lookup"><span data-stu-id="7c69c-177">Add constraint: Height, value: 15</span></span>
        - <span data-ttu-id="7c69c-178">Adicionar restrição: espaço à esquerda do visualização de conteúdo à margem esquerda, valor: 0</span><span class="sxs-lookup"><span data-stu-id="7c69c-178">Add constraint: Leading space to Content View Leading Margin, value: 0</span></span>
        - <span data-ttu-id="7c69c-179">Adicionar restrição: espaço à direita para a margem à direita do visualização de conteúdo, valor: 0</span><span class="sxs-lookup"><span data-stu-id="7c69c-179">Add constraint: Trailing space to Content View Trailing Margin, value: 0</span></span>
        - <span data-ttu-id="7c69c-180">Restrição Adicionar: Espaço superior para o Rótulo do Organizador Inferior, valor: Padrão</span><span class="sxs-lookup"><span data-stu-id="7c69c-180">Add constraint: Top space to Organizer Label Bottom, value: Standard</span></span>
        - <span data-ttu-id="7c69c-181">Adicionar restrição: Espaço inferior à Margem Inferior do Exibição de Conteúdo, valor: 0</span><span class="sxs-lookup"><span data-stu-id="7c69c-181">Add constraint: Bottom space to Content View Bottom Margin, value: 0</span></span>

1. <span data-ttu-id="7c69c-182">Select the **EventCell**, then select the **Size Inspector**.</span><span class="sxs-lookup"><span data-stu-id="7c69c-182">Select the **EventCell**, then select the **Size Inspector**.</span></span> <span data-ttu-id="7c69c-183">**Habilitar Automático** para **Altura da Linha.**</span><span class="sxs-lookup"><span data-stu-id="7c69c-183">Enable **Automatic** for **Row Height**.</span></span>

    ![Uma captura de tela dos controladores de exibição de calendário e tabela de calendário](images/calendar-table-storyboard.png)

1. <span data-ttu-id="7c69c-185">Abra **CalendarViewController.swift** e substitua seu conteúdo pelo código a seguir.</span><span class="sxs-lookup"><span data-stu-id="7c69c-185">Open **CalendarViewController.swift** and replace its contents with the following code.</span></span>

    :::code language="swift" source="../demo/GraphTutorial/GraphTutorial/CalendarViewController.swift" id="CalendarViewSnippet":::

1. <span data-ttu-id="7c69c-186">Execute o aplicativo, entre e toque na **guia** Calendário. Você deverá ver a lista de eventos.</span><span class="sxs-lookup"><span data-stu-id="7c69c-186">Run the app, sign in, and tap the **Calendar** tab. You should see the list of events.</span></span>

    ![Uma captura de tela da tabela de eventos](images/calendar-list.png)
