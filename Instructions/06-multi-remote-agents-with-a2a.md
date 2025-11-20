---
lab:
  title: A2A 프로토콜을 사용하여 원격 에이전트에 연결
  description: A2A 프로토콜을 사용하여 원격 에이전트와 공동 작업합니다.
---

# A2A 프로토콜을 사용하여 원격 에이전트에 연결

이 연습에서는 Azure AI 에이전트 서비스를 A2A 프로토콜과 함께 사용하여 서로 상호 작용하는 간단한 원격 에이전트를 만듭니다. 이러한 에이전트는 테크니컬 라이터가 개발자 블로그 게시물을 준비하도록 도와줍니다. 제목 에이전트는 헤드라인을 생성하고 개요 에이전트는 제목을 사용하여 문서에 대한 간략한 개요를 개발합니다. 이제 시작하겠습니다.

> **팁**: 이 연습에서 사용되는 코드는 Python용 Azure AI 파운드리를 기준으로 합니다. Microsoft .NET, JavaScript 및 Java용 SDK를 사용하여 유사한 솔루션을 개발할 수 있습니다. 자세한 내용은 [Azure AI 파운드리 SDK 클라이언트 라이브러리](https://learn.microsoft.com/azure/ai-foundry/how-to/develop/sdk-overview)를 참조하세요.

이 연습을 완료하는 데 약 **30**분 정도 소요됩니다.

> **참고**: 이 연습에 사용된 일부 기술은 미리 보기이거나 현재 개발 중에 있습니다. 예기치 않은 동작, 경고 또는 오류가 발생할 수 있습니다.

## Azure AI 파운드리 프로젝트 만들기

먼저 Azure AI 파운드리 프로젝트를 만들어 보겠습니다.

1. 웹 브라우저에서 [Azure AI 파운드리 포털](https://ai.azure.com)(`https://ai.azure.com`)을 열고 Azure 자격 증명을 사용하여 로그인합니다. 처음 로그인할 때 열리는 팁이나 빠른 시작 창을 닫고, 필요한 경우 왼쪽 위에 있는 **Azure AI 파운드리** 로고를 사용하여 다음 이미지와 유사한 홈페이지로 이동합니다(**도움말** 창이 열려 있는 경우 닫습니다).

    ![Azure AI Foundry 포털의 스크린샷.](./Media/ai-foundry-home.png)

1. 홈페이지에서 **에이전트 만들기**를 선택합니다.
1. 프로젝트를 만들라는 메시지가 표시되면 프로젝트의 유효한 이름을 입력하고 **고급 옵션**을 펼칩니다.
1. 프로젝트에 대한 다음 설정을 확인합니다.
    - **Azure AI 파운드리 리소스**: *Azure AI 파운드리 리소스의 유효한 이름*
    - **구독**: ‘Azure 구독’
    - **리소스 그룹**: ‘리소스 그룹 만들기 또는 선택’
    - **지역**: ***AI Foundry 권장 사항 선택***\*

    > \* 일부 Azure AI 리소스는 지역 모델 할당량에 의해 제한됩니다. 연습 후반부에 할당량 한도를 초과하는 경우 다른 지역에서 다른 리소스를 만들어야 할 수도 있습니다.

1. **만들기**를 선택한 다음, 프로젝트가 만들어질 때까지 기다립니다.
1. 메시지가 표시되면 할당량 가용성에 따라 *전역 표준* 또는 *표준* 배포 옵션을 사용하여 **gpt-4o** 모델을 배포합니다.

    >**참고**: 할당량을 사용할 수 있는 경우 에이전트 및 프로젝트를 만들 때 GPT-4o 기본 모델이 자동으로 배포될 수 있습니다.

1. 프로젝트를 만들면 에이전트 플레이그라운드가 열립니다.

1. 왼쪽 탐색 창에서 **개요**를 선택하면 다음과 같은 프로젝트의 메인 페이지가 표시됩니다.

    ![Azure AI 파운드리 프로젝트 개요 페이지의 스크린샷.](./Media/ai-foundry-project.png)

1. 클라이언트 응용 프로그램에서 프로젝트에 연결하는 데 사용하므로 **Azure AI 파운드리 프로젝트 엔드포인트** 값을 메모장에 복사합니다.

## A2A 애플리케이션 만들기

이제 에이전트를 사용하는 클라이언트 앱을 만들 준비가 되었습니다. GitHub 리포지토리에 일부 코드가 제공되었습니다.

### 애플리케이션 코드가 포함된 리포지토리 복제

1. 새 브라우저 탭을 엽니다(Azure AI 파운드리 포털을 기존 탭에서 열어 두기). 그런 다음 새 탭에서 [Azure Portal](https://portal.azure.com)(`https://portal.azure.com`)을 열고 메시지가 나타나면 Azure 자격 증명을 사용하여 로그인합니다.

    Azure Portal 홈페이지를 보려면 환영 알림을 닫습니다.

1. 페이지 상단의 검색 창 오른쪽에 있는 **[\>_]** 단추를 사용하여 Azure Portal에서 새 Cloud Shell을 만들고 구독에 저장소가 없는 ***PowerShell*** 환경을 선택합니다.

    Cloud Shell은 Azure Portal 하단의 창에서 명령줄 인터페이스를 제공합니다. 보다 쉽게 작업할 수 있도록 이 창의 크기를 조정하거나 최대화할 수 있습니다.

    > **참고**: 이전에 *Bash* 환경을 사용하는 Cloud Shell을 만든 경우 ***PowerShell***로 전환합니다.

1. Cloud Shell 도구 모음의 **설정** 메뉴에서 **클래식 버전으로 이동**을 선택합니다(코드 편집기를 사용하는 데 필요).

    **<font color="red">계속하기 전에 Cloud Shell의 클래식 버전으로 전환했는지 확인합니다.</font>**

1. Cloud Shell 창에서 다음 명령을 입력하여 이 연습의 코드 파일이 포함된 GitHub 리포지토리를 복제합니다(명령을 입력하거나 클립보드에 복사한 다음 명령줄을 마우스 오른쪽 단추로 클릭하여 일반 텍스트로 붙여넣습니다).

    ```
   rm -r ai-agents -f
   git clone https://github.com/MicrosoftLearning/mslearn-ai-agents ai-agents
    ```

    > **팁**: CloudShell에 명령을 입력할 때 출력이 화면 버퍼를 많이 차지하여 현재 줄의 커서가 가려질 수 있습니다. `cls` 명령을 입력해 화면을 지우면 각 작업에 더 집중할 수 있습니다.

1. 다음 명령을 입력하여 작업 디렉터리를 코드 파일이 포함된 폴더로 변경하고 모두 나열합니다.

    ```
   cd ai-agents/Labfiles/06-build-remote-agents-with-a2a/python
   ls -a -l
    ```

    제공된 파일에는 다음이 포함됩니다.
    ```output
    python
    ├── outline_agent/
    │   ├── agent.py
    │   ├── agent_executor.py
    │   └── server.py
    ├── routing_agent/
    │   ├── agent.py
    │   └── server.py
    ├── title_agent/
    │   ├── agent.py
    |   ├── agent_executor.py
    │   └── server.py
    ├── client.py
    └── run_all.py
    ```

    각 에이전트 폴더에는 Azure AI 에이전트 코드와 에이전트를 호스트할 서버가 포함되어 있습니다. **라우팅 에이전트**는 **제목** 및 **개요** 에이전트를 검색하고 통신합니다. **클라이언트**를 사용하여 라우팅 에이전트에 프롬프트를 제출할 수 있습니다. `run_all.py`는 모든 서버를 시작하고 클라이언트를 실행합니다.

### 애플리케이션 설정 구성

1. Cloud Shell 명령줄 창에서 다음 명령을 입력하여 사용할 라이브러리를 설치합니다.

    ```
   python -m venv labenv
   ./labenv/bin/Activate.ps1
   pip install -r requirements.txt azure-ai-projects azure-ai-agents a2a-sdk
    ```

1. 제공된 구성 파일을 편집하려면 다음 명령을 입력합니다.

    ```
   code .env
    ```

    코드 편집기에서 파일이 열립니다.

1. 코드 파일에서 **your_project_endpoint** 자리 표시자를 프로젝트의 엔드포인트(Azure AI Foundry 포털의 프로젝트 **개요** 페이지에서 복사됨)로 바꾸고 MODEL_DEPLOYMENT_NAME 변수가 모델 배포 이름(*gpt-4o*)으로 설정되어 있는지 확인합니다.
1. 자리 표시자를 바꾼 후에는 **CTRL+S** 명령을 사용하여 변경 내용을 저장한 다음 **CTRL+Q** 명령을 사용하여 Cloud Shell 명령줄을 열어둔 상태에서 코드 편집기를 닫습니다.

### 검색 가능한 에이전트 만들기

이 작업에서는 작성자가 문서에 대한 인기 있는 헤드라인을 만드는 데 도움이 되는 제목 에이전트를 만듭니다. 또한 A2A 프로토콜에서 에이전트를 검색 가능하게 만드는 데 필요한 에이전트의 기술 및 카드를 정의합니다.

1. `title_agent` 디렉터리로 이동합니다.

    ```
   cd title_agent
    ```

> **팁**: 코드를 추가할 때 올바른 들여쓰기를 유지해야 합니다. 주석 들여쓰기 수준을 가이드로 사용합니다.

1. 제공된 코드 파일을 편집하려면 다음 명령을 입력합니다.

    ```
   code agent.py
    ```

1. **Create the agents client** 주석을 찾아 다음 코드를 추가하여 Azure AI 프로젝트에 연결합니다.

    > **팁**: 들여쓰기 수준을 올바르게 유지하도록 주의합니다.

    ```python
   # Create the agents client
   self.client = AgentsClient(
       endpoint=os.environ['PROJECT_ENDPOINT'],
       credential=DefaultAzureCredential(
           exclude_environment_credential=True,
           exclude_managed_identity_credential=True
       )
   )
    ```

1. **Create the title agent** 주석을 찾아 다음 코드를 추가하여 에이전트를 만듭니다.

    ```python
   # Create the title agent
   self.agent = self.client.create_agent(
       model=os.environ['MODEL_DEPLOYMENT_NAME'],
       name='title-agent',
       instructions="""
       You are a helpful writing assistant.
       Given a topic the user wants to write about, suggest a single clear and catchy blog post title.
       """,
   )
    ```

1. **Create the title agent** 주석을 찾아 다음 코드를 추가하여 채팅 스레드를 만듭니다.

    ```python
   # Create a thread for the chat session
   thread = self.client.threads.create()
    ```

1. **Send user message** 주석을 찾아 이 코드를 추가하여 사용자의 프롬프트를 제출합니다.

    ```python
   # Send user message
   self.client.messages.create(thread_id=thread.id, role=MessageRole.USER, content=user_message)
    ```

1. **Create and run the agent** 주석을 찾아 다음 코드를 추가하여 에이전트의 응답 생성을 시작합니다.

    ```python
   # Create and run the agent
   run = self.client.runs.create_and_process(thread_id=thread.id, agent_id=self.agent.id)
    ```

    파일의 나머지 부분에 제공된 코드는 에이전트의 응답을 처리하고 반환합니다. 

1. 코드 파일을 저장합니다(*Ctrl+S*). 이제 에이전트의 기술 및 카드를 A2A 프로토콜과 공유할 준비가 되었습니다. 

1. 다음 명령을 입력하여 제목 에이전트의 `server.py` 파일을 편집합니다.  

    ```
   code server.py
    ```

1. **Define agent skills** 주석을 찾아 다음 코드를 추가하여 에이전트의 기능을 지정합니다.

    ```python
   # Define agent skills
   skills = [
       AgentSkill(
           id='generate_blog_title',
           name='Generate Blog Title',
           description='Generates a blog title based on a topic',
           tags=['title'],
           examples=[
               'Can you give me a title for this article?',
           ],
       ),
   ]
    ```

1. **Create agent card** 주석을 찾아 이 코드를 추가하여 에이전트를 검색 가능하게 만드는 메타데이터를 정의합니다.

    ```python
   # Create agent card
   agent_card = AgentCard(
       name='AI Foundry Title Agent',
       description='An intelligent title generator agent powered by Azure AI Foundry. '
       'I can help you generate catchy titles for your articles.',
       url=f'http://{host}:{port}/',
       version='1.0.0',
       default_input_modes=['text'],
       default_output_modes=['text'],
       capabilities=AgentCapabilities(),
       skills=skills,
   )
    ```

1. **Create agent executor** 주석을 찾아 다음 코드를 추가하여 에이전트 카드로 에이전트 실행기를 초기화합니다.

    ```python
   # Create agent executor
   agent_executor = create_foundry_agent_executor(agent_card)
    ```

    에이전트 실행기는 생성된 제목 에이전트의 래퍼 역할을 합니다.

1. **Create request handler** 주석을 찾아 다음을 추가하여 실행기를 사용하여 들어오는 요청을 처리합니다.

    ```python
   # Create request handler
   request_handler = DefaultRequestHandler(
       agent_executor=agent_executor, task_store=InMemoryTaskStore()
   )
    ```

1. **Create A2A application** 주석 아래에 이 코드를 추가하여 A2A 호환 애플리케이션 인스턴스를 만듭니다.

    ```python
   # Create A2A application
   a2a_app = A2AStarletteApplication(
       agent_card=agent_card, http_handler=request_handler
   )
    ```
    
    이 코드는 제목 에이전트의 정보를 공유하고 제목 에이전트 실행기를 사용하여 이 에이전트에 대한 들어오는 요청을 처리하는 A2A 서버를 만듭니다.

1. 완료되면 코드 파일(*Ctrl+S*)을 저장합니다.

### 에이전트 간에 메시지 사용

이 작업에서는 A2A 프로토콜을 사용하여 라우팅 에이전트가 다른 에이전트에 메시지를 보낼 수 있도록 합니다. 또한 에이전트 실행기 클래스를 구현하여 제목 에이전트가 메시지를 수신하도록 허용합니다.

1. `routing_agent` 디렉터리로 이동합니다.

    ```
   cd ../routing_agent
    ```

1. 제공된 코드 파일을 편집하려면 다음 명령을 입력합니다.

    ```
   code agent.py
    ```

    라우팅 에이전트는 사용자 메시지를 처리하고 요청을 처리해야 하는 원격 에이전트를 결정하는 오케스트레이터 역할을 합니다.

    사용자 메시지가 수신되면 라우팅 에이전트는 다음과 같이 합니다.
    - 대화 스레드를 시작합니다.
    - `create_and_process` 메서드를 사용하여 사용자 메시지에 가장 일치하는 에이전트를 평가합니다.
    - 메시지는 `send_message` 함수를 사용하여 HTTP를 통해 적절한 에이전트로 라우팅됩니다.
    - 원격 에이전트는 메시지를 처리하고 응답을 반환합니다.

    라우팅 에이전트는 마지막으로 응답을 캡처하고 스레드를 통해 사용자에게 반환합니다.

    `send_message` 메서드는 비동기적이며 에이전트 실행이 성공적으로 완료될 때까지 기다려야 합니다.

1. **Retrieve the remote agent's A2A client using the agent name** 주석 아래에 다음 코드를 추가합니다.

    ```python
   # Retrieve the remote agent's A2A client using the agent name 
   client = self.remote_agent_connections[agent_name]
    ```

1. **Construct the payload to send to the remote agent** 주석을 찾아 다음 코드를 추가합니다.

    ```python
   # Construct the payload to send to the remote agent
   payload: dict[str, Any] = {
       'message': {
           'role': 'user',
           'parts': [{'kind': 'text', 'text': task}],
           'messageId': message_id,
       },
   }
    ```

1. **Wrap the payload in a SendMessageRequest object** 주석을 찾아 다음 코드를 추가합니다.

    ```python
   # Wrap the payload in a SendMessageRequest object
   message_request = SendMessageRequest(id=message_id, params=MessageSendParams.model_validate(payload))
    ```

1. **Send the message to the remote agent client and await the response** 주석 아래에 다음 코드를 추가합니다.

    ```python
   # Send the message to the remote agent client and await the response
   send_response: SendMessageResponse = await client.send_message(message_request=message_request)
    ```


1. 완료되면 코드 파일(*Ctrl+S*)을 저장합니다. 이제 라우팅 에이전트가 메시지를 검색하여 제목 에이전트에 보낼 수 있습니다. 라우팅 에이전트에서 들어오는 메시지를 처리하기 위한 에이전트 실행기 코드를 만들어 보겠습니다.

1. `title_agent` 디렉터리로 이동합니다.

    ```
   cd ../title_agent
    ```

1. 제공된 코드 파일을 편집하려면 다음 명령을 입력합니다.

    ```
   code agent_executor.py
    ```

    `AgentExecutor` 클래스 구현에는 `execute` 및 `cancel` 메서드가 포함되어 있어야 합니다. cancel 메서드가 제공되었습니다. `execute` 메서드에는 작업이 완료되면 이벤트를 관리하고 발신자에게 신호를 보내는 개체가 `TaskUpdater` 포함되어 있습니다. 작업 실행을 위한 논리를 추가해 보겠습니다.

1. `execute` 메서드에서 **Process the request** 주석 아래에 다음 코드를 추가합니다.

    ```python
   # Process the request
   await self._process_request(context.message.parts, context.context_id, updater)
    ```

1. `_process_request` 메서드에서 **Get the title agent** 주석 아래에 다음 코드를 추가합니다.

    ```python
   # Get the title agent
   agent = await self._get_or_create_agent()
    ```

1. **Update the task status** 주석 아래에 다음 코드를 추가합니다.

    ```python
   # Update the task status
   await task_updater.update_status(
       TaskState.working,
       message=new_agent_text_message('Title Agent is processing your request...', context_id=context_id),
   )
    ```

1. **Run the agent conversation** 주석을 찾아 다음 코드를 추가합니다.

    ```python
   # Run the agent conversation
   responses = await agent.run_conversation(user_message)
    ```

1. **Update the task with the responses** 주석을 찾아 다음 코드를 추가합니다.

    ```python
   # Update the task with the responses
   for response in responses:
       await task_updater.update_status(
           TaskState.working,
           message=new_agent_text_message(response, context_id=context_id),
       )
    ```

1. **Mark the task as complete** 주석을 찾아 다음 코드를 추가합니다.

    ```python
   # Mark the task as complete
   final_message = responses[-1] if responses else 'Task completed.'
   await task_updater.complete(
       message=new_agent_text_message(final_message, context_id=context_id)
   )
    ```

    이제 제목 에이전트가 A2A 프로토콜에서 메시지를 처리하는 데 사용할 에이전트 실행기로 래핑되었습니다. 잘했습니다!

### Azure에 로그인하고 앱 실행

1. Cloud Shell 명령줄 창에서 다음 명령을 입력하여 Azure에 로그인합니다.

    ```
    az login
    ```

    **<font color="red">Cloud Shell 세션이 이미 인증되었더라도 Azure에 로그인해야 합니다.</font>**

    > **참고**: 대부분의 시나리오에서는 *az login*을 사용하는 것만으로도 충분합니다. 그러나 여러 테넌트에 구독이 있는 경우 *--tenant* 매개 변수를 사용하여 테넌트 지정해야 할 수 있습니다. 자세한 내용은 [Sign into Azure interactively using the Azure CLI](https://learn.microsoft.com/cli/azure/authenticate-azure-cli-interactively)를 참조하세요.
    
1. 메시지가 표시되면 지침에 따라 새 탭에서 로그인 페이지를 열고 제공된 인증 코드와 Azure 자격 증명을 입력합니다. 그런 다음 명령줄에서 로그인 프로세스를 완료하고 메시지가 표시되면 Azure AI 파운드리 허브가 포함된 구독을 선택합니다.
1. 로그인한 후 다음 명령을 입력하여 애플리케이션을 실행합니다.

    ```
    cd ..
    python run_all.py
    ```
    
    애플리케이션은 인증된 Azure 세션에 대한 자격 증명을 사용하여 실행하고 프로젝트에 연결한 후 에이전트를 만들고 실행합니다. 각 서버가 시작될 때 일부 출력이 표시됩니다.

1. 입력 프롬프트가 나타날 때까지 기다린 후 다음과 같은 프롬프트를 입력합니다.

    ```
   Create a title and outline for an article about React programming.
    ```

    잠시 후 에이전트의 응답이 결과와 함께 표시됩니다.

1. `quit` 키를 눌러 프로그램을 종료하고 서버를 중지합니다.
    
## 요약

이 연습에서는 Azure AI 에이전트 서비스 SDK 및 A2A Python SDK를 사용하여 원격 다중 에이전트 솔루션을 만들었습니다. 검색 가능한 A2A 호환 에이전트를 만들고 에이전트의 기술에 액세스하도록 라우팅 에이전트를 설정했습니다. 또한 들어오는 A2A 메시지를 처리하고 작업을 관리하는 에이전트 실행기를 구현했습니다. 잘했습니다!

## 정리

Azure AI 에이전트 서비스 탐색을 마친 경우 이 연습에서 만든 리소스를 삭제하여 불필요한 Azure 비용이 발생하지 않도록 해야 합니다.

1. Azure Portal이 포함된 브라우저 탭으로 돌아가서(또는 새 브라우저 탭의 `https://portal.azure.com`에서 [Azure Portal](https://portal.azure.com)을 다시 열고) 이 연습에 사용된 리소스를 배포한 리소스 그룹의 콘텐츠를 확인합니다.
1. 도구 모음에서 **리소스 그룹 삭제**를 선택합니다.
1. 리소스 그룹 이름을 입력하고 삭제할 것인지 확인합니다.
