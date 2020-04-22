<!-- markdownlint-disable MD002 MD041 -->

<span data-ttu-id="2908b-101">Neste exercício, você incorporará o Microsoft Graph no aplicativo.</span><span class="sxs-lookup"><span data-stu-id="2908b-101">In this exercise you will incorporate the Microsoft Graph into the application.</span></span> <span data-ttu-id="2908b-102">Para este aplicativo, você usará o [SDK do Microsoft Graph para o objetivo C](https://github.com/microsoftgraph/msgraph-sdk-objc) para fazer chamadas para o Microsoft Graph.</span><span class="sxs-lookup"><span data-stu-id="2908b-102">For this application, you will use the [Microsoft Graph SDK for Objective C](https://github.com/microsoftgraph/msgraph-sdk-objc) to make calls to Microsoft Graph.</span></span>

## <a name="get-calendar-events-from-outlook"></a><span data-ttu-id="2908b-103">Obter eventos de calendário do Outlook</span><span class="sxs-lookup"><span data-stu-id="2908b-103">Get calendar events from Outlook</span></span>

<span data-ttu-id="2908b-104">Nesta seção, você estenderá a `GraphManager` classe para adicionar uma função para obter os eventos e a atualização `CalendarViewController` do usuário para usar essas novas funções.</span><span class="sxs-lookup"><span data-stu-id="2908b-104">In this section you will extend the `GraphManager` class to add a function to get the user's events and update `CalendarViewController` to use these new functions.</span></span>

1. <span data-ttu-id="2908b-105">Abra o **graphmanager. Swift** e adicione o método a seguir `GraphManager` à classe.</span><span class="sxs-lookup"><span data-stu-id="2908b-105">Open **GraphManager.swift** and add the following method to the `GraphManager` class.</span></span>

    ```Swift
    public func getEvents(completion: @escaping(Data?, Error?) -> Void) {
        // GET /me/events?$select='subject,organizer,start,end'$orderby=createdDateTime DESC
        // Only return these fields in results
        let select = "$select=subject,organizer,start,end"
        // Sort results by when they were created, newest first
        let orderBy = "$orderby=createdDateTime+DESC"
        let eventsRequest = NSMutableURLRequest(url: URL(string: "\(MSGraphBaseURL)/me/events?\(select)&\(orderBy)")!)
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
    > <span data-ttu-id="2908b-106">Considere o que o código `getEvents` está fazendo.</span><span class="sxs-lookup"><span data-stu-id="2908b-106">Consider what the code in `getEvents` is doing.</span></span>
    >
    > - <span data-ttu-id="2908b-107">A URL que será chamada é `/v1.0/me/events`.</span><span class="sxs-lookup"><span data-stu-id="2908b-107">The URL that will be called is `/v1.0/me/events`.</span></span>
    > - <span data-ttu-id="2908b-108">O `select` parâmetro de consulta limita os campos retornados para cada evento para apenas aqueles que o aplicativo usará realmente.</span><span class="sxs-lookup"><span data-stu-id="2908b-108">The `select` query parameter limits the fields returned for each events to just those the app will actually use.</span></span>
    > - <span data-ttu-id="2908b-109">O `orderby` parâmetro de consulta classifica os resultados pela data e hora em que foram criados, com o item mais recente em primeiro lugar.</span><span class="sxs-lookup"><span data-stu-id="2908b-109">The `orderby` query parameter sorts the results by the date and time they were created, with the most recent item being first.</span></span>

1. <span data-ttu-id="2908b-110">Abra **CalendarViewController. Swift** e substitua todo o conteúdo pelo código a seguir.</span><span class="sxs-lookup"><span data-stu-id="2908b-110">Open **CalendarViewController.swift** and replace its entire contents with the following code.</span></span>

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

            GraphManager.instance.getEvents {
                (data: Data?, error: Error?) in
                DispatchQueue.main.async {
                    self.spinner.stop()

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

<span data-ttu-id="2908b-111">Agora você pode executar o aplicativo, entrar e tocar no item de navegação de **calendário** no menu.</span><span class="sxs-lookup"><span data-stu-id="2908b-111">You can now run the app, sign in, and tap the **Calendar** navigation item in the menu.</span></span> <span data-ttu-id="2908b-112">Você deve ver um despejo JSON dos eventos no aplicativo.</span><span class="sxs-lookup"><span data-stu-id="2908b-112">You should see a JSON dump of the events in the app.</span></span>

## <a name="display-the-results"></a><span data-ttu-id="2908b-113">Exibir os resultados</span><span class="sxs-lookup"><span data-stu-id="2908b-113">Display the results</span></span>

<span data-ttu-id="2908b-114">Agora você pode substituir o despejo JSON por algo para exibir os resultados de forma amigável.</span><span class="sxs-lookup"><span data-stu-id="2908b-114">Now you can replace the JSON dump with something to display the results in a user-friendly manner.</span></span> <span data-ttu-id="2908b-115">Nesta seção, você irá modificar a `getEvents` função para retornar objetos fortemente tipados e modificar `CalendarViewController` para usar um modo de exibição de tabela para renderizar os eventos.</span><span class="sxs-lookup"><span data-stu-id="2908b-115">In this section, you will modify the `getEvents` function to return strongly-typed objects, and modify `CalendarViewController` to use a table view to render the events.</span></span>

1. <span data-ttu-id="2908b-116">Abra o **graphmanager. Swift**.</span><span class="sxs-lookup"><span data-stu-id="2908b-116">Open **GraphManager.swift**.</span></span> <span data-ttu-id="2908b-117">Substitua a função `getEvents` existente pelo seguinte.</span><span class="sxs-lookup"><span data-stu-id="2908b-117">Replace the existing `getEvents` function with the following.</span></span>

    :::code language="swift" source="../demo/GraphTutorial/GraphTutorial/GraphManager.swift" id="GetEventsSnippet" highlight="1,17-38":::

1. <span data-ttu-id="2908b-118">Crie um novo arquivo de **classe Touch** do Cocoa **GraphTutorial** no projeto GraphTutorial `CalendarTableViewCell.swift`chamado.</span><span class="sxs-lookup"><span data-stu-id="2908b-118">Create a new **Cocoa Touch Class** file in the **GraphTutorial** project named `CalendarTableViewCell.swift`.</span></span> <span data-ttu-id="2908b-119">Escolha **UITableViewCell** na **subclasse de** Field.</span><span class="sxs-lookup"><span data-stu-id="2908b-119">Choose **UITableViewCell** in the **Subclass of** field.</span></span>

1. <span data-ttu-id="2908b-120">Abra **CalendarTableViewCell. Swift** e adicione o código a seguir à `CalendarTableViewCell` classe.</span><span class="sxs-lookup"><span data-stu-id="2908b-120">Open **CalendarTableViewCell.swift** and add the following code to the `CalendarTableViewCell` class.</span></span>

    :::code language="swift" source="../demo/GraphTutorial/GraphTutorial/CalendarTableViewCell.swift" id="PropertiesSnippet":::

1. <span data-ttu-id="2908b-121">Abra **Main. Storyboard** e localize a **cena do calendário**.</span><span class="sxs-lookup"><span data-stu-id="2908b-121">Open **Main.storyboard** and locate the **Calendar Scene**.</span></span> <span data-ttu-id="2908b-122">Selecione o **modo de exibição** na **cena do calendário** e exclua-o.</span><span class="sxs-lookup"><span data-stu-id="2908b-122">Select the **View** in the **Calendar Scene** and delete it.</span></span>

    ![Uma captura de tela do modo de exibição da cena de calendário](./images/view-in-calendar-scene.png)

1. <span data-ttu-id="2908b-124">Adicione um **modo de exibição de tabela** da **biblioteca** à **cena do calendário**.</span><span class="sxs-lookup"><span data-stu-id="2908b-124">Add a **Table View** from the **Library** to the **Calendar Scene**.</span></span>
1. <span data-ttu-id="2908b-125">Selecione o modo de exibição de tabela e, em seguida, selecione o **Inspetor de atributos**.</span><span class="sxs-lookup"><span data-stu-id="2908b-125">Select the table view, then select the **Attributes Inspector**.</span></span> <span data-ttu-id="2908b-126">Definir **células de protótipo** como **1**.</span><span class="sxs-lookup"><span data-stu-id="2908b-126">Set **Prototype Cells** to **1**.</span></span>
1. <span data-ttu-id="2908b-127">Use a **biblioteca** para adicionar três **Rótulos** à célula prototype.</span><span class="sxs-lookup"><span data-stu-id="2908b-127">Use the **Library** to add three **Labels** to the prototype cell.</span></span>
1. <span data-ttu-id="2908b-128">Selecione a célula Prototype e, em seguida, selecione o **Inspetor de identidade**.</span><span class="sxs-lookup"><span data-stu-id="2908b-128">Select the prototype cell, then select the **Identity Inspector**.</span></span> <span data-ttu-id="2908b-129">Altere a **classe** para **CalendarTableViewCell**.</span><span class="sxs-lookup"><span data-stu-id="2908b-129">Change **Class** to **CalendarTableViewCell**.</span></span>
1. <span data-ttu-id="2908b-130">Selecione o **Inspetor de atributos** e defina o `EventCell` **identificador** como.</span><span class="sxs-lookup"><span data-stu-id="2908b-130">Select the **Attributes Inspector** and set **Identifier** to `EventCell`.</span></span>
1. <span data-ttu-id="2908b-131">Com o **EventCell** selecionado, selecione o **Inspetor de conexões** e `durationLabel`Conecte `organizerLabel`-se `subjectLabel` , e para os rótulos que você adicionou à célula no storyboard.</span><span class="sxs-lookup"><span data-stu-id="2908b-131">With the **EventCell** selected, select the **Connections Inspector** and connect `durationLabel`, `organizerLabel`, and `subjectLabel` to the labels you added to the cell on the storyboard.</span></span>
1. <span data-ttu-id="2908b-132">Defina as propriedades e as restrições nos três rótulos da seguinte maneira.</span><span class="sxs-lookup"><span data-stu-id="2908b-132">Set the properties and constraints on the three labels as follows.</span></span>

    - <span data-ttu-id="2908b-133">**Rótulo de assunto**</span><span class="sxs-lookup"><span data-stu-id="2908b-133">**Subject Label**</span></span>
        - <span data-ttu-id="2908b-134">Adicionar restrição: espaço à esquerda na margem esquerda da exibição de conteúdo, valor: 0</span><span class="sxs-lookup"><span data-stu-id="2908b-134">Add constraint: Leading space to Content View Leading Margin, value: 0</span></span>
        - <span data-ttu-id="2908b-135">Adicionar restrição: espaço à direita à margem à direita da exibição do conteúdo, valor: 0</span><span class="sxs-lookup"><span data-stu-id="2908b-135">Add constraint: Trailing space to Content View Trailing Margin, value: 0</span></span>
        - <span data-ttu-id="2908b-136">Adicionar restrição: espaço superior à margem superior do modo de exibição de conteúdo, valor: 0</span><span class="sxs-lookup"><span data-stu-id="2908b-136">Add constraint: Top space to Content View Top Margin, value: 0</span></span>
    - <span data-ttu-id="2908b-137">**Rótulo do organizador**</span><span class="sxs-lookup"><span data-stu-id="2908b-137">**Organizer Label**</span></span>
        - <span data-ttu-id="2908b-138">Fonte: System 12,0</span><span class="sxs-lookup"><span data-stu-id="2908b-138">Font: System 12.0</span></span>
        - <span data-ttu-id="2908b-139">Adicionar restrição: espaço à esquerda na margem esquerda da exibição de conteúdo, valor: 0</span><span class="sxs-lookup"><span data-stu-id="2908b-139">Add constraint: Leading space to Content View Leading Margin, value: 0</span></span>
        - <span data-ttu-id="2908b-140">Adicionar restrição: espaço à direita à margem à direita da exibição do conteúdo, valor: 0</span><span class="sxs-lookup"><span data-stu-id="2908b-140">Add constraint: Trailing space to Content View Trailing Margin, value: 0</span></span>
        - <span data-ttu-id="2908b-141">Adicionar restrição: espaço superior para rótulo de assunto inferior, valor: padrão</span><span class="sxs-lookup"><span data-stu-id="2908b-141">Add constraint: Top space to Subject Label Bottom, value: Standard</span></span>
    - <span data-ttu-id="2908b-142">**Rótulo de duração**</span><span class="sxs-lookup"><span data-stu-id="2908b-142">**Duration Label**</span></span>
        - <span data-ttu-id="2908b-143">Fonte: System 12,0</span><span class="sxs-lookup"><span data-stu-id="2908b-143">Font: System 12.0</span></span>
        - <span data-ttu-id="2908b-144">Cor: cor cinza escuro</span><span class="sxs-lookup"><span data-stu-id="2908b-144">Color: Dark Gray Color</span></span>
        - <span data-ttu-id="2908b-145">Adicionar restrição: espaço à esquerda na margem esquerda da exibição de conteúdo, valor: 0</span><span class="sxs-lookup"><span data-stu-id="2908b-145">Add constraint: Leading space to Content View Leading Margin, value: 0</span></span>
        - <span data-ttu-id="2908b-146">Adicionar restrição: espaço à direita à margem à direita da exibição do conteúdo, valor: 0</span><span class="sxs-lookup"><span data-stu-id="2908b-146">Add constraint: Trailing space to Content View Trailing Margin, value: 0</span></span>
        - <span data-ttu-id="2908b-147">Adicionar restrição: espaço superior ao rótulo do organizador inferior, valor: padrão</span><span class="sxs-lookup"><span data-stu-id="2908b-147">Add constraint: Top space to Organizer Label Bottom, value: Standard</span></span>
        - <span data-ttu-id="2908b-148">Adicionar restrição: espaço inferior à margem inferior da visualização de conteúdo, valor: 8</span><span class="sxs-lookup"><span data-stu-id="2908b-148">Add constraint: Bottom space to Content View Bottom Margin, value: 8</span></span>

    ![Uma captura de tela do layout da célula de protótipo](./images/prototype-cell-layout.png)

1. <span data-ttu-id="2908b-150">Abra **CalendarViewController. Swift** e substitua seu conteúdo pelo código a seguir.</span><span class="sxs-lookup"><span data-stu-id="2908b-150">Open **CalendarViewController.swift** and replace its contents with the following code.</span></span>

    :::code language="swift" source="../demo/GraphTutorial/GraphTutorial/CalendarViewController.swift" id="CalendarViewSnippet":::

1. <span data-ttu-id="2908b-151">Execute o aplicativo, entre e toque na guia **calendário** . Você deve ver a lista de eventos.</span><span class="sxs-lookup"><span data-stu-id="2908b-151">Run the app, sign in, and tap the **Calendar** tab. You should see the list of events.</span></span>

    ![Uma captura de tela da tabela de eventos](./images/calendar-list.png)
