<!-- markdownlint-disable MD002 MD041 -->

Nesta seção, você adicionará a capacidade de criar eventos no calendário do usuário.

1. Abra **GraphManager.swift** e adicione a função a seguir para criar um novo evento no calendário do usuário.

    :::code language="swift" source="../demo/GraphTutorial/GraphTutorial/GraphManager.swift" id="CreateEventSnippet":::

1. Crie um novo **arquivo de classe Cocoa Touch** na pasta **GraphTutorial** chamada `NewEventViewController` . Escolha **UIViewController** na **Subclasse do** campo.
1. Abra **NewEventViewController.swift** e substitua seu conteúdo pelo seguinte.

    :::code language="swift" source="../demo/GraphTutorial/GraphTutorial/NewEventViewController.swift" id="NewEventViewControllerSnippet":::

1. Abra **Main.storyboard**. Use a **Biblioteca para** arrastar um Controlador **de Exibição** para o storyboard.
1. Usando a **Biblioteca,** adicione uma **Barra de Navegação** ao controlador de exibição.
1. Clique duas vezes no **título** na barra de navegação e atualize-o para `New Event` .
1. Usando a **biblioteca**, adicione um **item de** botão de barra ao lado esquerdo da barra de navegação.
1. Selecione o novo botão de barra e selecione o **Inspetor de Atributos.** Altere **o título** para `Cancel` .
1. Usando a **biblioteca**, adicione um **item de** botão de barra ao lado direito da barra de navegação.
1. Selecione o novo botão de barra e selecione o **Inspetor de Atributos.** Altere **o título** para `Create` .
1. Selecione o controlador de exibição e, em seguida, selecione o **Inspetor de Identidade.** Altere **a** classe **para NewEventViewController**.
1. Adicione os seguintes controles da **Biblioteca** à exibição.

    - Adicione um **Rótulo** na barra de navegação. Definir seu texto como `Subject` .
    - Adicione um **campo de** texto sob o rótulo. De definir **seu atributo placeholder** como `Subject` .
    - Adicione um **Rótulo** sob o campo de texto. Definir seu texto como `Attendees` .
    - Adicione um **campo de** texto sob o rótulo. De definir **seu atributo placeholder** como `Separate multiple entries with ;` .
    - Adicione um **Rótulo** sob o campo de texto. Definir seu texto como `Start` .
    - Adicione um **Selador de Data** sob o rótulo. Definir seu **estilo preferencial** para **compactar**, **seu intervalo** como **15 minutos** e sua altura como **35**.
    - Adicione um **Rótulo** no selador de data. Definir seu texto como `End` .
    - Adicione um **Selador de Data** sob o rótulo. Definir seu **estilo preferencial** para **compactar**, **seu intervalo** como **15 minutos** e sua altura como **35**.
    - Adicione um **ponto de exibição de** texto no selador de data.

1. Selecione o **Novo Controlador de Exibição de Evento** e use o Inspetor de **Conexão** para fazer as seguintes conexões.

    - Conecte a **ação de** cancelamento recebida ao **botão Cancelar** barra.
    - Conecte a **ação createEvent** recebida ao **botão Criar** barra.
    - Conecte a **saída do** assunto ao primeiro campo de texto.
    - Conecte a **saída dos participantes** ao segundo campo de texto.
    - Conecte a **saída** de início ao primeiro se picker de data.
    - Conecte a **saída final** ao segundo se picker de data.
    - Conecte a **saída do** corpo ao exibição de texto.

1. Adicione as restrições a seguir.

    - **Barra de Navegação**
        - Espaço à frente para a Área de Segurança, valor: 0
        - Espaço à trailing para Área de Segurança, valor: 0
        - Espaço superior para Área de Segurança, valor: 0
        - Altura, valor: 44
    - **Rótulo de Assunto**
        - Espaço à esquerda para a margem exibir, valor: 0
        - Espaço à direita para Exibir margem, valor: 0
        - Espaço superior para a Barra de Navegação, valor: 20
    - **Campo Texto do Assunto**
        - Espaço à esquerda para a margem exibir, valor: 0
        - Espaço à direita para Exibir margem, valor: 0
        - Espaço superior para o Rótulo de Assunto, valor: Padrão
    - **Rótulo dos participantes**
        - Espaço à esquerda para a margem exibir, valor: 0
        - Espaço à direita para Exibir margem, valor: 0
        - Espaço superior para o Campo de Texto do Assunto, valor: Padrão
    - **Campo de texto Participantes**
        - Espaço à esquerda para a margem exibir, valor: 0
        - Espaço à direita para Exibir margem, valor: 0
        - Espaço superior para o Rótulo de Participantes, valor: Padrão
    - **Rótulo iniciar**
        - Espaço à esquerda para a margem exibir, valor: 0
        - Espaço à direita para Exibir margem, valor: 0
        - Espaço superior para o Campo de Texto do Assunto, valor: Padrão
    - **Start Date Picker**
        - Espaço à esquerda para a margem exibir, valor: 0
        - Espaço à direita para Exibir margem, valor: 0
        - Espaço superior para o Rótulo de Participantes, valor: Padrão
        - Altura, valor: 35
    - **Rótulo final**
        - Espaço à esquerda para a margem exibir, valor: 0
        - Espaço à direita para Exibir margem, valor: 0
        - Espaço superior para o Se picker de data de início, valor: Padrão
    - **End Date Picker**
        - Espaço à esquerda para a margem exibir, valor: 0
        - Espaço à direita para Exibir margem, valor: 0
        - Espaço superior para Rótulo Final, valor: Padrão
        - Altura: 35
    - **Exibição de Corpo de Texto**
        - Espaço à esquerda para a margem exibir, valor: 0
        - Espaço à direita para Exibir margem, valor: 0
        - Espaço superior para o Selador de Data de Término, valor: Padrão
        - Espaço inferior para Exibir margem, valor: 0

    ![Uma captura de tela do novo formulário de eventos no storyboard](images/new-event-form.png)

1. Selecione a **Cena do Calendário** e selecione o Inspetor de **Conexões.**
1. Em **Segues Disparados,** arraste o círculo não preenchido ao lado de **manual** para o Novo Controlador de Modo de Exibição **de** Eventos no storyboard. Selecione **Apresentar Modalmente** no menu pop-up.
1. Selecione o segue que você acabou de adicionar e selecione o **Inspetor de Atributos.** De definir **o campo Identificador** como `showEventForm` .
1. Conecte a **ação showNewEventForm** recebida ao botão **+** da barra de navegação.
1. Salve suas alterações e reinicie o aplicativo. Vá para a página de calendário e toque no **+** botão. Preencha o formulário e toque em **Criar** para criar um novo evento.

    ![Uma captura de tela do novo formulário de eventos](images/create-event.png)
