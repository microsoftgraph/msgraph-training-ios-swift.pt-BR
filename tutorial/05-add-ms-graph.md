<!-- markdownlint-disable MD002 MD041 -->

Neste exercício, você incorporará o Microsoft Graph no aplicativo. Para este aplicativo, você usará o [SDK do Microsoft Graph para o objetivo C](https://github.com/microsoftgraph/msgraph-sdk-objc) para fazer chamadas para o Microsoft Graph.

## <a name="get-calendar-events-from-outlook"></a>Obter eventos de calendário do Outlook

Nesta seção, você estenderá a `GraphManager` classe para adicionar uma função para obter os eventos e a atualização `CalendarViewController` do usuário para usar essas novas funções.

1. Abra o **graphmanager. Swift** e adicione o método a seguir `GraphManager` à classe.

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
    > Considere o que o código `getEvents` está fazendo.
    >
    > - A URL que será chamada é `/v1.0/me/events`.
    > - O `select` parâmetro de consulta limita os campos retornados para cada evento para apenas aqueles que o aplicativo usará realmente.
    > - O `orderby` parâmetro de consulta classifica os resultados pela data e hora em que foram criados, com o item mais recente em primeiro lugar.

1. Abra **CalendarViewController. Swift** e substitua todo o conteúdo pelo código a seguir.

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

Agora você pode executar o aplicativo, entrar e tocar no item de navegação de **calendário** no menu. Você deve ver um despejo JSON dos eventos no aplicativo.

## <a name="display-the-results"></a>Exibir os resultados

Agora você pode substituir o despejo JSON por algo para exibir os resultados de forma amigável. Nesta seção, você irá modificar a `getEvents` função para retornar objetos fortemente tipados e modificar `CalendarViewController` para usar um modo de exibição de tabela para renderizar os eventos.

### <a name="update-getevents"></a>Atualizar GetEvents

1. Abra o **graphmanager. Swift**. Altere a `getEvents` declaração de função para o seguinte.

    ```Swift
    public func getEvents(completion: @escaping([MSGraphEvent]?, Error?) -> Void)
    ```

1. Substitua a linha `completion(eventsData, nil)` pelo código a seguir.

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

### <a name="update-calendarviewcontroller"></a>Atualizar CalendarViewController

1. Crie um novo arquivo de **classe Touch** do Cocoa **** no projeto GraphTutorial `CalendarTableViewCell.swift`chamado. Escolha **UITableViewCell** na **subclasse de** Field.
1. Abra **CalendarTableViewCell. Swift** e adicione o código a seguir à `CalendarTableViewCell` classe.

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

1. Abra **Main. Storyboard** e localize a **cena do calendário**. Selecione o **modo de exibição** na **cena do calendário** e exclua-o.

    ![Uma captura de tela do modo de exibição da cena de calendário](./images/view-in-calendar-scene.png)

1. Adicione um **modo de exibição de tabela** da **biblioteca** à **cena do calendário**.
1. Selecione o modo de exibição de tabela e, em seguida, selecione o **Inspetor de atributos**. Definir **células de protótipo** como **1**.
1. Use a **biblioteca** para adicionar três **Rótulos** à célula prototype.
1. Selecione a célula Prototype e, em seguida, selecione o **Inspetor de identidade**. Altere a **classe** para **CalendarTableViewCell**.
1. Selecione o **Inspetor de atributos** e defina o `EventCell` **identificador** como.
1. Com o **EventCell** selecionado, selecione o **Inspetor de conexões** e `durationLabel`Conecte `organizerLabel`-se `subjectLabel` , e para os rótulos que você adicionou à célula no storyboard.

    ![Uma captura de tela do layout da célula de protótipo](./images/prototype-cell-layout.png)

1. Abra **CalendarViewController. Swift** e substitua seu conteúdo pelo código a seguir.

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

1. Execute o aplicativo, entre e toque na guia **calendário** . Você deve ver a lista de eventos.

    ![Uma captura de tela da tabela de eventos](./images/calendar-list.png)
