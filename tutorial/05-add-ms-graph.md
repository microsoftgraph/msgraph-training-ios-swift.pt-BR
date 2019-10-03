<!-- markdownlint-disable MD002 MD041 -->

<span data-ttu-id="5b4e6-101">Neste exercício, você incorporará o Microsoft Graph no aplicativo.</span><span class="sxs-lookup"><span data-stu-id="5b4e6-101">In this exercise you will incorporate the Microsoft Graph into the application.</span></span> <span data-ttu-id="5b4e6-102">Para este aplicativo, você usará o [SDK do Microsoft Graph para o objetivo C](https://github.com/microsoftgraph/msgraph-sdk-objc) para fazer chamadas para o Microsoft Graph.</span><span class="sxs-lookup"><span data-stu-id="5b4e6-102">For this application, you will use the [Microsoft Graph SDK for Objective C](https://github.com/microsoftgraph/msgraph-sdk-objc) to make calls to Microsoft Graph.</span></span>

## <a name="get-calendar-events-from-outlook"></a><span data-ttu-id="5b4e6-103">Obter eventos de calendário do Outlook</span><span class="sxs-lookup"><span data-stu-id="5b4e6-103">Get calendar events from Outlook</span></span>

<span data-ttu-id="5b4e6-104">Nesta seção, você estenderá a `GraphManager` classe para adicionar uma função para obter os eventos e a atualização `CalendarViewController` do usuário para usar essas novas funções.</span><span class="sxs-lookup"><span data-stu-id="5b4e6-104">In this section you will extend the `GraphManager` class to add a function to get the user's events and update `CalendarViewController` to use these new functions.</span></span>

1. <span data-ttu-id="5b4e6-105">Abra o **graphmanager. Swift** e adicione o método a seguir `GraphManager` à classe.</span><span class="sxs-lookup"><span data-stu-id="5b4e6-105">Open **GraphManager.swift** and add the following method to the `GraphManager` class.</span></span>

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
    > <span data-ttu-id="5b4e6-106">Considere o que o código `getEvents` está fazendo.</span><span class="sxs-lookup"><span data-stu-id="5b4e6-106">Consider what the code in `getEvents` is doing.</span></span>
    >
    > - <span data-ttu-id="5b4e6-107">A URL que será chamada é `/v1.0/me/events`.</span><span class="sxs-lookup"><span data-stu-id="5b4e6-107">The URL that will be called is `/v1.0/me/events`.</span></span>
    > - <span data-ttu-id="5b4e6-108">O `select` parâmetro de consulta limita os campos retornados para cada evento para apenas aqueles que o aplicativo usará realmente.</span><span class="sxs-lookup"><span data-stu-id="5b4e6-108">The `select` query parameter limits the fields returned for each events to just those the app will actually use.</span></span>
    > - <span data-ttu-id="5b4e6-109">O `orderby` parâmetro de consulta classifica os resultados pela data e hora em que foram criados, com o item mais recente em primeiro lugar.</span><span class="sxs-lookup"><span data-stu-id="5b4e6-109">The `orderby` query parameter sorts the results by the date and time they were created, with the most recent item being first.</span></span>

1. <span data-ttu-id="5b4e6-110">Abra **CalendarViewController. Swift** e substitua todo o conteúdo pelo código a seguir.</span><span class="sxs-lookup"><span data-stu-id="5b4e6-110">Open **CalendarViewController.swift** and replace its entire contents with the following code.</span></span>

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

<span data-ttu-id="5b4e6-111">Agora você pode executar o aplicativo, entrar e tocar no item de navegação de **calendário** no menu.</span><span class="sxs-lookup"><span data-stu-id="5b4e6-111">You can now run the app, sign in, and tap the **Calendar** navigation item in the menu.</span></span> <span data-ttu-id="5b4e6-112">Você deve ver um despejo JSON dos eventos no aplicativo.</span><span class="sxs-lookup"><span data-stu-id="5b4e6-112">You should see a JSON dump of the events in the app.</span></span>

## <a name="display-the-results"></a><span data-ttu-id="5b4e6-113">Exibir os resultados</span><span class="sxs-lookup"><span data-stu-id="5b4e6-113">Display the results</span></span>

<span data-ttu-id="5b4e6-114">Agora você pode substituir o despejo JSON por algo para exibir os resultados de forma amigável.</span><span class="sxs-lookup"><span data-stu-id="5b4e6-114">Now you can replace the JSON dump with something to display the results in a user-friendly manner.</span></span> <span data-ttu-id="5b4e6-115">Nesta seção, você irá modificar a `getEvents` função para retornar objetos fortemente tipados e modificar `CalendarViewController` para usar um modo de exibição de tabela para renderizar os eventos.</span><span class="sxs-lookup"><span data-stu-id="5b4e6-115">In this section, you will modify the `getEvents` function to return strongly-typed objects, and modify `CalendarViewController` to use a table view to render the events.</span></span>

### <a name="update-getevents"></a><span data-ttu-id="5b4e6-116">Atualizar GetEvents</span><span class="sxs-lookup"><span data-stu-id="5b4e6-116">Update getEvents</span></span>

1. <span data-ttu-id="5b4e6-117">Abra o **graphmanager. Swift**.</span><span class="sxs-lookup"><span data-stu-id="5b4e6-117">Open **GraphManager.swift**.</span></span> <span data-ttu-id="5b4e6-118">Altere a `getEvents` declaração de função para o seguinte.</span><span class="sxs-lookup"><span data-stu-id="5b4e6-118">Change the `getEvents` function declaration to the following.</span></span>

    ```Swift
    public func getEvents(completion: @escaping([MSGraphEvent]?, Error?) -> Void)
    ```

1. <span data-ttu-id="5b4e6-119">Substitua a linha `completion(eventsData, nil)` pelo código a seguir.</span><span class="sxs-lookup"><span data-stu-id="5b4e6-119">Replace the `completion(eventsData, nil)` line with the following code.</span></span>

    ```Swift
    do {
        // Deserialize response as events collection
        let eventsCollection = try MSCollection(data: eventsData)
        var eventArray: [MSGraphEvent] = []

        eventsCollection.value.forEach({
            (rawEvent: Any) in
            // Convert JSON to a dictionary
            guard let eventDict = rawEvent as? [String: Any] else {
                return
            }

            // Deserialize event from the dictionary
            let event = MSGraphEvent(dictionary: eventDict)!
            eventArray.append(event)
        })

        // Return the array
        completion(eventArray, nil)
    } catch {
        completion(nil, error)
    }
    ```

### <a name="update-calendarviewcontroller"></a><span data-ttu-id="5b4e6-120">Atualizar CalendarViewController</span><span class="sxs-lookup"><span data-stu-id="5b4e6-120">Update CalendarViewController</span></span>

1. <span data-ttu-id="5b4e6-121">Crie um novo arquivo de **classe Touch** do Cocoa \*\*\*\* no projeto GraphTutorial `CalendarTableViewCell.swift`chamado.</span><span class="sxs-lookup"><span data-stu-id="5b4e6-121">Create a new **Cocoa Touch Class** file in the **GraphTutorial** project named `CalendarTableViewCell.swift`.</span></span> <span data-ttu-id="5b4e6-122">Escolha **UITableViewCell** na **subclasse de** Field.</span><span class="sxs-lookup"><span data-stu-id="5b4e6-122">Choose **UITableViewCell** in the **Subclass of** field.</span></span>
1. <span data-ttu-id="5b4e6-123">Abra **CalendarTableViewCell. Swift** e adicione o código a seguir à `CalendarTableViewCell` classe.</span><span class="sxs-lookup"><span data-stu-id="5b4e6-123">Open **CalendarTableViewCell.swift** and add the following code to the `CalendarTableViewCell` class.</span></span>

    ```Swift
    @IBOutlet var subjectLabel: UILabel!
    @IBOutlet var organizerLabel: UILabel!
    @IBOutlet var durationLabel: UILabel!

    var subject: String? {
        didSet {
            subjectLabel.text = subject
        }
    }

    var organizer: String? {
        didSet {
            organizerLabel.text = organizer
        }
    }

    var duration: String? {
        didSet {
            durationLabel.text = duration
        }
    }
    ```

1. <span data-ttu-id="5b4e6-124">Abra **Main. Storyboard** e localize a **cena do calendário**.</span><span class="sxs-lookup"><span data-stu-id="5b4e6-124">Open **Main.storyboard** and locate the **Calendar Scene**.</span></span> <span data-ttu-id="5b4e6-125">Selecione o **modo de exibição** na **cena do calendário** e exclua-o.</span><span class="sxs-lookup"><span data-stu-id="5b4e6-125">Select the **View** in the **Calendar Scene** and delete it.</span></span>

    ![Uma captura de tela do modo de exibição da cena de calendário](./images/view-in-calendar-scene.png)

1. <span data-ttu-id="5b4e6-127">Adicione um **modo de exibição de tabela** da **biblioteca** à **cena do calendário**.</span><span class="sxs-lookup"><span data-stu-id="5b4e6-127">Add a **Table View** from the **Library** to the **Calendar Scene**.</span></span>
1. <span data-ttu-id="5b4e6-128">Selecione o modo de exibição de tabela e, em seguida, selecione o **Inspetor de atributos**.</span><span class="sxs-lookup"><span data-stu-id="5b4e6-128">Select the table view, then select the **Attributes Inspector**.</span></span> <span data-ttu-id="5b4e6-129">Definir **células de protótipo** como **1**.</span><span class="sxs-lookup"><span data-stu-id="5b4e6-129">Set **Prototype Cells** to **1**.</span></span>
1. <span data-ttu-id="5b4e6-130">Use a **biblioteca** para adicionar três **Rótulos** à célula prototype.</span><span class="sxs-lookup"><span data-stu-id="5b4e6-130">Use the **Library** to add three **Labels** to the prototype cell.</span></span>
1. <span data-ttu-id="5b4e6-131">Selecione a célula Prototype e, em seguida, selecione o **Inspetor de identidade**.</span><span class="sxs-lookup"><span data-stu-id="5b4e6-131">Select the prototype cell, then select the **Identity Inspector**.</span></span> <span data-ttu-id="5b4e6-132">Altere a **classe** para **CalendarTableViewCell**.</span><span class="sxs-lookup"><span data-stu-id="5b4e6-132">Change **Class** to **CalendarTableViewCell**.</span></span>
1. <span data-ttu-id="5b4e6-133">Selecione o **Inspetor de atributos** e defina o `EventCell` **identificador** como.</span><span class="sxs-lookup"><span data-stu-id="5b4e6-133">Select the **Attributes Inspector** and set **Identifier** to `EventCell`.</span></span>
1. <span data-ttu-id="5b4e6-134">Com o **EventCell** selecionado, selecione o **Inspetor de conexões** e `durationLabel`Conecte `organizerLabel`-se `subjectLabel` , e para os rótulos que você adicionou à célula no storyboard.</span><span class="sxs-lookup"><span data-stu-id="5b4e6-134">With the **EventCell** selected, select the **Connections Inspector** and connect `durationLabel`, `organizerLabel`, and `subjectLabel` to the labels you added to the cell on the storyboard.</span></span>

    ![Uma captura de tela do layout da célula de protótipo](./images/prototype-cell-layout.png)

1. <span data-ttu-id="5b4e6-136">Abra **CalendarViewController. Swift** e substitua seu conteúdo pelo código a seguir.</span><span class="sxs-lookup"><span data-stu-id="5b4e6-136">Open **CalendarViewController.swift** and replace its contents with the following code.</span></span>

    ```Swift
    import UIKit
    import MSGraphClientModels

    class CalendarViewController: UITableViewController {

        private let tableCellIdentifier = "EventCell"
        private let spinner = SpinnerViewController()
        private var events: [MSGraphEvent]?

        override func viewDidLoad() {
            super.viewDidLoad()

            // Do any additional setup after loading the view.
            self.spinner.start(container: self)

            GraphManager.instance.getEvents {
                (eventArray: [MSGraphEvent]?, error: Error?) in
                DispatchQueue.main.async {
                    self.spinner.stop()

                    guard let events = eventArray, error == nil else {
                        // Show the error
                        let alert = UIAlertController(title: "Error getting events",
                                                      message: error.debugDescription,
                                                      preferredStyle: .alert)

                        alert.addAction(UIAlertAction(title: "OK", style: .default, handler: nil))
                        self.present(alert, animated: true)
                        return
                    }

                    self.events = events
                    self.tableView.reloadData()
                }
            }
        }

        // Number of sections, always 1
        override func numberOfSections(in tableView: UITableView) -> Int {
            return 1
        }

        // Return the number of events in the table
        override func tableView(_ tableView: UITableView, numberOfRowsInSection section: Int) -> Int {
            return events?.count ?? 0
        }

        override func tableView(_ tableView: UITableView, cellForRowAt indexPath: IndexPath) -> UITableViewCell {
            let cell = tableView.dequeueReusableCell(withIdentifier: tableCellIdentifier, for: indexPath) as! CalendarTableViewCell

            // Get the event that corresponds to the row
            let event = events?[indexPath.row]

            // Configure the cell
            cell.subject = event?.subject
            cell.organizer = event?.organizer?.emailAddress?.name

            // Build a duration string
            let duration = "\(self.formatGraphDateTime(dateTime: event?.start)) to \(self.formatGraphDateTime(dateTime: event?.end))"
            cell.duration = duration

            return cell
        }

        private func formatGraphDateTime(dateTime: MSGraphDateTimeTimeZone?) -> String {
            guard let graphDateTime = dateTime else {
                return ""
            }

            // Create a formatter to parse Graph's date format
            let isoFormatter = DateFormatter()
            isoFormatter.dateFormat = "yyyy-MM-dd'T'HH:mm:ss.SSSSSSS"

            // Specify the timezone
            isoFormatter.timeZone = TimeZone(identifier: graphDateTime.timeZone!)

            let date = isoFormatter.date(from: graphDateTime.dateTime)

            // Output like 5/5/2019, 2:00 PM
            let dateFormatter = DateFormatter()
            dateFormatter.dateStyle = .short
            dateFormatter.timeStyle = .short

            return dateFormatter.string(from: date!)
        }
    }
    ```

1. <span data-ttu-id="5b4e6-137">Execute o aplicativo, entre e toque na guia **calendário** . Você deve ver a lista de eventos.</span><span class="sxs-lookup"><span data-stu-id="5b4e6-137">Run the app, sign in, and tap the **Calendar** tab. You should see the list of events.</span></span>

    ![Uma captura de tela da tabela de eventos](./images/calendar-list.png)
