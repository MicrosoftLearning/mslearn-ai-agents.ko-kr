---
lab:
  title: AI 에이전트 배포
  description: Azure AI 에이전트 서비스를 사용하여 기본 제공 도구를 사용하는 에이전트를 개발합니다.
---

# AI 에이전트 배포

이 연습에서는 Azure AI 에이전트 서비스를 사용하여 데이터를 분석하고 차트를 만드는 간단한 에이전트를 만듭니다. 에이전트는 기본 제공 *코드 인터프리터* 도구를 사용하여 차트를 이미지로 만드는 데 필요한 코드를 동적으로 생성한 다음 결과 차트 이미지를 저장합니다.

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
    - **지역**: **AI 서비스 지원 위치 *선택***\*

    > \* 일부 Azure AI 리소스는 지역 모델 할당량에 의해 제한됩니다. 연습 후반부에 할당량 한도를 초과하는 경우 다른 지역에서 다른 리소스를 만들어야 할 수도 있습니다.

1. **만들기**를 선택한 다음, 프로젝트가 만들어질 때까지 기다립니다.
1. 프로젝트가 생성되면 에이전트 플레이그라운드가 자동으로 열리므로 모델을 선택하거나 배포할 수 있습니다.

    ![Azure AI 파운드리 프로젝트 에이전트 플레이그라운드의 스크린샷.](./Media/ai-foundry-agents-playground.png)

    >**참고**: 에이전트 및 프로젝트를 만들 때 GPT-4o 베이스 모델이 자동으로 배포됩니다.

1. 왼쪽 탐색 창에서 **개요**를 선택하면 다음과 같은 프로젝트의 메인 페이지가 표시됩니다.

    > **참고**: *권한 부족** 오류가 표시되면 **수정** 버튼을 사용하여 문제를 해결합니다.

    ![Azure AI 파운드리 프로젝트 개요 페이지의 스크린샷.](./Media/ai-foundry-project.png)

1. 클라이언트 응용 프로그램에서 프로젝트에 연결하는 데 사용하므로 **Azure AI 파운드리 프로젝트 엔드포인트** 값을 메모장에 복사합니다.

## 에이전트 클라이언트 앱 만들기

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
   cd ai-agents/Labfiles/02-build-ai-agent/Python
   ls -a -l
    ```

    제공된 파일에는 애플리케이션 코드, 구성 설정 및 데이터가 포함됩니다.

### 애플리케이션 설정 구성

1. Cloud Shell 명령줄 창에서 다음 명령을 입력하여 사용할 라이브러리를 설치합니다.

    ```
   python -m venv labenv
   ./labenv/bin/Activate.ps1
   pip install -r requirements.txt azure-ai-projects
    ```

1. 제공된 구성 파일을 편집하려면 다음 명령을 입력합니다.

    ```
   code .env
    ```

    코드 편집기에서 파일이 열립니다.

1. 코드 파일에서 **your_project_endpoint** 자리 표시자를 프로젝트의 엔드포인트로 바꿉니다(Azure AI 파운드리 포털의 프로젝트 **개요** 페이지에서 복사).
1. 자리 표시자를 바꾼 후에는 **CTRL+S** 명령을 사용하여 변경 내용을 저장한 다음 **CTRL+Q** 명령을 사용하여 Cloud Shell 명령줄을 열어둔 상태에서 코드 편집기를 닫습니다.

### 에이전트 앱에 대한 코드 작성

> **팁**: 코드를 추가할 때 올바른 들여쓰기를 유지해야 합니다. 주석 들여쓰기 수준을 가이드로 사용합니다.

1. 제공된 코드 파일을 편집하려면 다음 명령을 입력합니다.

    ```
   code agent.py
    ```

1. 애플리케이션 구성 설정을 검색하고 분석할 *data.txt*의 데이터를 로드하는 기존 코드를 검토합니다. 파일의 나머지 부분에는 데이터 분석 에이전트를 구현하는 데 필요한 코드를 추가하는 주석이 포함되어 있습니다.
1. **Add references** 주석을 찾고 다음 코드를 추가하여 기본 제공 코드 인터프리터 도구를 사용하는 Azure AI 에이전트를 빌드하는 데 필요한 클래스를 가져옵니다.

    ```python
   # Add references
   from azure.identity import DefaultAzureCredential
   from azure.ai.agents import AgentsClient
   from azure.ai.agents.models import FilePurpose, CodeInterpreterTool, ListSortOrder, MessageRole
    ```

1. **Connect to the Agent client** 주석을 찾아 다음 코드를 추가하여 Azure AI 프로젝트에 연결합니다.

    > **팁**: 들여쓰기 수준을 올바르게 유지하도록 주의합니다.

    ```python
   # Connect to the Agent client
   agent_client = AgentsClient(
       endpoint=project_endpoint,
       credential=DefaultAzureCredential
           (exclude_environment_credential=True,
            exclude_managed_identity_credential=True)
   )
   with agent_client:
    ```

    이 코드는 현재 Azure 자격 증명을 사용하여 Azure AI 파운드리 프로젝트에 연결합니다. 마지막 *with agent_client* 문은 클라이언트의 범위를 정의하는 코드 블록을 시작하여 블록 내의 코드가 완료되면 정리되도록 합니다.

1. *with agent_client* 블록 내에서 **Upload the data file and create a CodeInterpreterTool** 주석을 찾아 다음 코드를 추가하여 데이터 파일을 프로젝트에 업로드하고 그 안에 있는 데이터에 액세스할 수 있는 CodeInterpreterTool을 만듭니다.

    ```python
   # Upload the data file and create a CodeInterpreterTool
   file = agent_client.files.upload_and_poll(
        file_path=file_path, purpose=FilePurpose.AGENTS
   )
   print(f"Uploaded {file.filename}")

   code_interpreter = CodeInterpreterTool(file_ids=[file.id])
    ```
    
1. **Define an agent that uses the CodeInterpreterTool** 주석을 찾고 다음 코드를 추가하여 데이터를 분석하고 이전에 정의한 코드 인터프리터 도구를 사용할 수 있는 AI 에이전트를 정의합니다.

    ```python
   # Define an agent that uses the CodeInterpreterTool
   agent = agent_client.create_agent(
        model=model_deployment,
        name="data-agent",
        instructions="You are an AI agent that analyzes the data in the file that has been uploaded. If the user requests a chart, create it and save it as a .png file.",
        tools=code_interpreter.definitions,
        tool_resources=code_interpreter.resources,
   )
   print(f"Using agent: {agent.name}")
    ```

1. **Create a thread for the conversation** 주석을 찾고 다음 코드를 추가하여 에이전트와의 채팅 세션이 실행되는 스레드를 시작합니다.

    ```python
   # Create a thread for the conversation
   thread = agent_client.threads.create()
    ```
    
1. 코드의 다음 섹션에서는 사용자가 프롬프트를 입력할 수 있도록 루프를 설정하고 사용자가 "quit"을 입력하면 종료됩니다.

1. **Send a prompt to the agent** 주석을 찾고 다음 코드를 추가하여 (이전에 로드된 파일의 데이터와 함께) 프롬프트에 사용자 메시지를 추가한 다음 에이전트와 함께 스레드를 실행합니다.

    ```python
   # Send a prompt to the agent
   message = agent_client.messages.create(
        thread_id=thread.id,
        role="user",
        content=user_prompt,
    )

   run = agent_client.runs.create_and_process(thread_id=thread.id, agent_id=agent.id)
     
   # Check the run status for failures
   if run.status == "failed":
        print(f"Run failed: {run.last_error}")
    ```

1. **Show the latest response from the agent** 주석을 찾고 다음 코드를 추가하여 완료된 스레드에서 메시지를 검색하고 에이전트가 보낸 마지막 메시지를 표시합니다.

    ```python
   # Show the latest response from the agent
   last_msg = agent_client.messages.get_last_message_text_by_role(
       thread_id=thread.id,
       role=MessageRole.AGENT,
   )
   if last_msg:
       print(f"Last Message: {last_msg.text.value}")
    ```

1. 루프가 종료된 후의 **Get the conversation history** 주석을 찾고 다음 코드를 추가하여 대화 스레드에서 메시지를 출력하되, 순서를 반대로 변경하여 시간순으로 표시합니다.

    ```python
   # Get the conversation history
   print("\nConversation Log:\n")
   messages = agent_client.messages.list(thread_id=thread.id, order=ListSortOrder.ASCENDING)
   for message in messages:
       if message.text_messages:
           last_msg = message.text_messages[-1]
           print(f"{message.role}: {last_msg.text.value}\n")
    ```

1. **Get any generated files** 주석을 찾고 다음 코드를 추가하여 (에이전트가 내부 스토리지에 파일을 저장했음을 나타내는) 메시지에서 파일 경로 주석을 가져온 다음 파일을 앱 폴더에 복사합니다. _참고_: 현재 시스템에서 이미지 콘텐츠를 사용할 수 없습니다.

    ```python
   # Get any generated files
   for msg in messages:
       # Save every image file in the message
       for img in msg.image_contents:
           file_id = img.image_file.file_id
           file_name = f"{file_id}_image_file.png"
           agent_client.files.save(file_id=file_id, file_name=file_name)
           print(f"Saved image file to: {Path.cwd() / file_name}")
    ```

1. **Clean up** 주석을 찾고 다음 코드를 추가하여 더 이상 필요하지 않은 경우 에이전트 및 스레드를 삭제합니다.

    ```python
   # Clean up
   agent_client.delete_agent(agent.id)
    ```

1. 주석을 사용하여 코드를 검토하고 다음 방법을 이해합니다.
    - AI 파운드리 프로젝트에 연결합니다.
    - 데이터 파일을 업로드하고 액세스할 수 있는 코드 인터프리터 도구를 만듭니다.
    - 코드 인터프리터 도구를 사용하고 데이터를 분석하고 차트를 .png 파일로 만드는 명시적 지침이 있는 새 에이전트를 만듭니다.
    - 분석할 데이터와 함께 사용자의 프롬프트 메시지를 사용하여 스레드를 실행합니다.
    - 오류가 발생할 경우 실행 상태를 확인합니다.
    - 완료된 스레드에서 메시지를 검색하고 에이전트가 보낸 마지막 메시지를 표시합니다.
    - 대화 내용을 표시합니다.
    - 생성된 각 파일을 저장합니다.
    - 더 이상 필요하지 않은 경우 에이전트와 스레드를 삭제합니다.

1. 완료되면 코드 파일(*Ctrl+S*)을 저장합니다. 또한 코드 편집기(*Ctrl+Q*)를 닫을 수도 있습니다. 하지만 추가한 코드를 편집해야 하는 경우를 대비해 계속 열어 두는 것이 좋습니다. 두 경우 모두 Cloud Shell 명령줄 창을 열어 둡니다.

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
    python agent.py
    ```
    
    애플리케이션은 인증된 Azure 세션에 대한 자격 증명을 사용하여 실행하고 프로젝트에 연결한 후 에이전트를 만들고 실행합니다.

1. 메시지가 표시되면 앱이 *data.txt* 텍스트 파일에서 로드한 데이터를 확인합니다. 다음 프롬프트 입력:

    ```
   What's the category with the highest cost?
    ```

    > **팁**: 속도 제한을 초과하여 앱이 실패하는 경우. 몇 초 정도 기다렸다가 다시 시도하세요. 구독에서 사용할 수 있는 할당량이 부족한 경우 모델이 응답하지 않을 수 있습니다.

1. 응답을 봅니다. 그런 다음, 이번에는 차트를 요청하는 다른 프롬프트를 입력합니다.

    ```
   Create a pie chart showing cost by category
    ```

    이 경우 에이전트는 필요에 따라 코드 인터프리터 도구를 선택적으로 사용하여 요청에 따라 차트를 만들어야 합니다.

1. 원하는 경우 대화를 계속할 수 있습니다. 스레드는 *상태 저장*이므로 대화 내용을 유지합니다. 즉, 에이전트에 각 응답에 대한 전체 컨텍스트가 있습니다. 완료되면 `quit`을(를) 입력합니다.
1. 스레드에서 검색된 대화 메시지와 생성된 파일을 검토합니다.

1. 애플리케이션이 완료되면 cloud shell **다운로드** 명령을 사용하여 앱 폴더에 저장된 각 .png 파일을 다운로드합니다. 예시:

    ```
   download ./<file_name>.png
    ```

    다운로드 명령은 브라우저의 오른쪽 아래에 팝업 링크를 만듭니다. 이 링크를 선택하여 파일을 다운로드하고 열 수 있습니다.

## 요약

이 연습에서는 Azure AI 에이전트 서비스 SDK를 사용하여 AI 에이전트를 사용하는 클라이언트 애플리케이션을 만들었습니다. 에이전트는 기본 제공 코드 인터프리터 도구를 사용하여 이미지를 만드는 동적 코드를 실행합니다.

## 정리

Azure AI 에이전트 서비스 탐색을 마친 경우 이 연습에서 만든 리소스를 삭제하여 불필요한 Azure 비용이 발생하지 않도록 해야 합니다.

1. Azure Portal이 포함된 브라우저 탭으로 돌아가서(또는 새 브라우저 탭의 `https://portal.azure.com`에서 [Azure Portal](https://portal.azure.com)을 다시 열고) 이 연습에 사용된 리소스를 배포한 리소스 그룹의 콘텐츠를 확인합니다.
1. 도구 모음에서 **리소스 그룹 삭제**를 선택합니다.
1. 리소스 그룹 이름을 입력하고 삭제할 것인지 확인합니다.
