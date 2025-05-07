---
lab:
  title: AI 에이전트에서 사용자 지정 함수 사용
  description: 함수를 사용하여 에이전트에 사용자 지정 기능을 추가하는 방법을 알아봅니다.
---

# AI 에이전트에서 사용자 지정 함수 사용

이 연습에서는 사용자 지정 함수를 도구로 사용하여 작업을 완료할 수 있는 에이전트를 만드는 방법을 살펴봅니다.

기술 문제에 대한 세부 정보를 수집하고 지원 티켓을 생성할 수 있는 간단한 기술 지원 에이전트를 빌드합니다.

이 연습을 완료하는 데 약 **30**분 정도 소요됩니다.

> **참고**: 이 연습에 사용된 일부 기술은 미리 보기이거나 현재 개발 중에 있습니다. 예기치 않은 동작, 경고 또는 오류가 발생할 수 있습니다.

## Azure AI 파운드리 프로젝트 만들기

먼저 Azure AI 파운드리 프로젝트를 만들어 보겠습니다.

1. 웹 브라우저에서 [Azure AI 파운드리 포털](https://ai.azure.com)(`https://ai.azure.com`)을 열고 Azure 자격 증명을 사용하여 로그인합니다. 처음 로그인할 때 열리는 팁이나 빠른 시작 창을 닫고, 필요한 경우 왼쪽 위에 있는 **Azure AI 파운드리** 로고를 사용하여 다음 이미지와 유사한 홈페이지로 이동합니다(**도움말** 창이 열려 있는 경우 닫습니다).

    ![Azure AI Foundry 포털의 스크린샷.](./Media/ai-foundry-home.png)

1. 홈페이지에서 **+ 프로젝트 만들기**를 선택합니다.
1. **프로젝트 만들기** 마법사에서 유효한 프로젝트 이름을 입력하고 기존 허브가 추천되면 새 허브를 만드는 옵션을 선택합니다. 그런 다음 허브 및 프로젝트를 지원하기 위해 자동으로 만들어지는 Azure 리소스를 검토합니다.
1. **사용자 지정**을 선택하고 허브에 대해 다음 설정을 지정합니다.
    - **허브 이름**: *허브에서 유효한 이름*
    - **구독**: ‘Azure 구독’
    - **리소스 그룹**: ‘리소스 그룹 만들기 또는 선택’
    - **위치**: 다음 지역 중 하나를 선택합니다.\*
        - eastus
        - eastus2
        - 스웨덴 중부
        - westus
        - westus3
    - **Azure AI 서비스 또는 Azure OpenAI 연결**: *새 AI 서비스 리소스 만들기*
    - **Azure AI 검색 연결**: 연결 건너뛰기

    > \* 작성 시 이러한 지역은 에이전트에서 사용할 gpt-4o 모델을 지원합니다. 모델 가용성은 지역 할당량에 의해 제한됩니다. 연습 후반부에 할당량 한도에 도달하는 경우 다른 지역에서 다른 프로젝트를 만들어야 할 수도 있습니다.

1. **다음**을 선택하여 구성을 검토합니다. **만들기**를 선택하고 프로세스가 완료될 때까지 기다립니다.
1. 프로젝트를 만들 때 표시되는 팁을 모두 닫고 Azure AI 파운드리 포털에서 프로젝트 페이지를 검토합니다. 이 페이지는 다음 이미지와 유사합니다.

    ![Azure AI 파운드리 포털의 Azure AI 프로젝트 세부 정보 스크린샷.](./Media/ai-foundry-project.png)

## 생성형 AI 모델 배포

이제 에이전트를 지원하기 위해 생성형 AI 언어 모델을 배포할 준비가 되었습니다.

1. 프로젝트 왼쪽 창의 **내 자산** 섹션에서 **모델 + 엔드포인트** 페이지를 선택합니다.
1. **모델 + 엔드포인트** 페이지의 **모델 배포** 탭의 **+ 모델 배포** 메뉴에서 **기본 모델 배포**를 선택합니다.
1. 목록에서 **gpt-4o** 모델을 검색하고 선택한 후 확인합니다.
1. 배포 세부 정보에서 **사용자 지정**을 선택하여 다음 설정으로 모델을 배포합니다.
    - **배포 이름**: *모델 배포에 대한 유효한 이름*
    - **배포 유형**: 글로벌 표준
    - **자동 버전 업데이트**: 사용
    - **모델 버전**: *사용 가능한 최신 버전 선택*
    - **연결된 AI 리소스**: *Azure OpenAI 리소스 연결 선택*
    - **분당 토큰 속도 제한(천 단위)**: 50K *(또는 50K 이하인 경우 구독에서 사용 가능한 최대치)*
    - **콘텐츠 필터**: DefaultV2

    > **참고**: TPM을 줄이면 사용 중인 구독에서 사용 가능한 할당량을 과도하게 사용하지 않을 수 있습니다. 이 연습에 사용되는 데이터는 50,000TPM이면 충분합니다. 사용 가능한 할당량이 이 수치 이하이면 연습을 완료할 수 있지만 속도 제한을 초과하는 경우 기다린 다음 프롬프트를 다시 제출해야 할 수 있습니다.

1. 배포가 완료될 때가지 기다립니다.

## 함수 도구를 사용하는 에이전트 개발

AI 파운드리에서 프로젝트를 만들었으므로 이제 사용자 지정 함수 도구를 사용하여 에이전트를 구현하는 앱을 개발해 보겠습니다.

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
   cd ai-agents/Labfiles/03-ai-agent-functions/Python
   ls -a -l
    ```

    제공된 파일에는 애플리케이션 코드와 구성 설정에 대한 파일이 포함됩니다.

### 애플리케이션 설정을 구성합니다.

1. Cloud Shell 명령줄 창에서 다음 명령을 입력하여 사용할 라이브러리를 설치합니다.

    ```
   python -m venv labenv
   ./labenv/bin/Activate.ps1
   pip install python-dotenv azure-identity azure-ai-projects
    ```

    >**참고:** 라이브러리 설치 중에 표시되는 경고 또는 오류 메시지를 무시할 수 있습니다.

1. 제공된 구성 파일을 편집하려면 다음 명령을 입력합니다.

    ```
   code .env
    ```

    코드 편집기에서 파일이 열립니다.

1. 코드 파일에서 **your_project_connection_string** 자리 표시자를 프로젝트의 연결 문자열(Azure AI 파운드리 포털의 프로젝트 **개요** 페이지에서 복사)로 바꾸고, **our_model_deployment** 자리 표시자를 GPT-4o 모델 배포에 할당한 이름으로 바꿉니다.
1. 자리 표시자를 바꾼 후 **Ctrl+S** 명령을 사용하여 변경 내용을 저장한 다음 **Ctrl+Q** 명령을 사용하여 Cloud Shell 명령줄을 열어 두고 코드 편집기를 닫습니다.

### 사용자 지정 함수 정의

1. 함수 코드용으로 제공된 코드 파일을 편집하려면 다음 명령을 입력합니다.

    ```
   code user_functions.py
    ```

1. **Create a function to submit a support ticket** 주석을 찾아 티켓 번호를 생성하고 지원 티켓을 텍스트 파일로 저장하는 다음 코드를 추가합니다.

    ```python
   # Create a function to submit a support ticket
   def submit_support_ticket(email_address: str, description: str) -> str:
        script_dir = Path(__file__).parent  # Get the directory of the script
        ticket_number = str(uuid.uuid4()).replace('-', '')[:6]
        file_name = f"ticket-{ticket_number}.txt"
        file_path = script_dir / file_name
        text = f"Support ticket: {ticket_number}\nSubmitted by: {email_address}\nDescription:\n{description}"
        file_path.write_text(text)
    
        message_json = json.dumps({"message": f"Support ticket {ticket_number} submitted. The ticket file is saved as {file_name}"})
        return message_json
    ```

1. **Define a set of callable functions** 주석을 찾아 이 코드 파일에 호출 가능한 함수 집합을 정적으로 정의하는 다음 코드를 추가합니다(이 경우에는 하나만 있지만 실제 솔루션에서는 에이전트가 호출할 수 있는 함수가 여러 개 있을 수 있습니다).

    ```python
   # Define a set of callable functions
   user_functions: Set[Callable[..., Any]] = {
        submit_support_ticket
    }
    ```
1. 파일을 저장합니다(*CTRL+S*).

### 함수를 사용할 수 있는 에이전트를 구현하는 코드 작성

1. 다음 명령을 입력하여 에이전트 코드 편집을 시작합니다.

    ```
    code agent.py
    ```

    > **팁**: 코드 파일에 코드를 추가할 때 올바른 들여쓰기를 유지해야 합니다.

1. 애플리케이션 구성 설정을 검색하고 사용자가 에이전트에 대한 프롬프트를 입력할 수 있는 루프를 설정하는 기존 코드를 검토합니다. 파일의 나머지 부분에는 기술 지원 에이전트를 구현하는 데 필요한 코드를 추가하는 주석이 포함되어 있습니다.
1. **Add references** 주석을 찾아 다음 코드를 추가하여 함수 코드를 도구로 사용하는 Azure AI 에이전트를 빌드하는 데 필요한 클래스를 가져옵니다.

    ```python
   # Add references
   from azure.identity import DefaultAzureCredential
   from azure.ai.projects import AIProjectClient
   from azure.ai.projects.models import FunctionTool, ToolSet
   from user_functions import user_functions
    ```

1. **Connect to the Azure AI Foundry project** 주석을 찾아 다음 코드를 추가하여 현재 Azure 자격 증명을 사용하여 Azure AI 프로젝트에 연결합니다.

    > **팁**: 들여쓰기 수준을 올바르게 유지하도록 주의하세요.

    ```python
   # Connect to the Azure AI Foundry project
   project_client = AIProjectClient.from_connection_string(
        credential=DefaultAzureCredential
            (exclude_environment_credential=True,
             exclude_managed_identity_credential=True),
        conn_str=PROJECT_CONNECTION_STRING
   )
    ```
    
1. **Define an agent that can use the custom functions** 섹션의 주석을 찾아 다음 코드를 추가하여 함수 코드를 도구 집합에 추가한 다음, 도구 집합을 사용할 수 있는 에이전트와 채팅 세션을 실행할 스레드를 만듭니다.

    ```python
   # Define an agent that can use the custom functions
   with project_client:

        functions = FunctionTool(user_functions)
        toolset = ToolSet()
        toolset.add(functions)
        project_client.agents.enable_auto_function_calls(toolset=toolset)
            
        agent = project_client.agents.create_agent(
            model=MODEL_DEPLOYMENT,
            name="support-agent",
            instructions="""You are a technical support agent.
                            When a user has a technical issue, you get their email address and a description of the issue.
                            Then you use those values to submit a support ticket using the function available to you.
                            If a file is saved, tell the user the file name.
                         """,
            toolset=toolset
        )

        thread = project_client.agents.create_thread()
        print(f"You're chatting with: {agent.name} ({agent.id})")

    ```

1. **Send a prompt to the agent** 주석을 찾아 다음 코드를 추가하여 사용자의 프롬프트를 메시지로 추가하고 스레드를 실행합니다.

    ```python
   # Send a prompt to the agent
   message = project_client.agents.create_message(
        thread_id=thread.id,
        role="user",
        content=user_prompt
   )
   run = project_client.agents.create_and_process_run(thread_id=thread.id, agent_id=agent.id)
    ```

    > **참고**: 스레드를 실행할 때 **create_and_process_run** 메서드를 사용하면, 에이전트가 함수의 이름과 매개변수를 기반으로 해당 함수들을 자동으로 찾아 사용할 수 있습니다. 대안으로 **create_run** 메서드를 사용할 수 있는데, 이 경우 함수 호출이 필요한 시점을 결정하기 위해 실행 상태를 폴링하는 코드를 작성하여 함수를 호출하고 그 결과를 에이전트에 반환할 책임이 있습니다.

1. **Check the run status for failures** 주석을 찾아 다음 코드를 추가하여 발생하는 오류를 표시합니다.

    ```python
   # Check the run status for failures
   if run.status == "failed":
        print(f"Run failed: {run.last_error}")
    ```

1. **Show the latest response from the agent** 주석을 찾아 다음 코드를 추가하여 완료된 스레드에서 메시지를 검색하고 에이전트가 보낸 마지막 메시지를 표시합니다.

    ```python
   # Show the latest response from the agent
   messages = project_client.agents.list_messages(thread_id=thread.id)
   last_msg = messages.get_last_text_message_by_role("assistant")
   if last_msg:
        print(f"Last Message: {last_msg.text.value}")
    ```

1. 반복문이 끝난 후에 있는 **Get the conversation history** 주석을 찾은 다음, 다음 코드를 추가하여 대화 스레드의 메시지를 출력합니다. 이때 메시지들은 시간 순으로 표시되도록 순서를 반대로 정렬합니다.

    ```python
   # Get the conversation history
   print("\nConversation Log:\n")
   messages = project_client.agents.list_messages(thread_id=thread.id)
   for message_data in reversed(messages.data):
        last_message_content = message_data.content[-1]
        print(f"{message_data.role}: {last_message_content.text.value}\n")
    ```

1. **Clean up** 주석을 찾아 다음 코드를 추가하여 더 이상 필요하지 않은 에이전트와 스레드를 삭제합니다.

    ```python
   # Clean up
   project_client.agents.delete_agent(agent.id)
   project_client.agents.delete_thread(thread.id)
    ```

1. 주석을 사용하여 코드를 검토하고 방법을 이해합니다.
    - 도구 집합에 사용자 지정 함수 집합 추가
    - 도구 집합을 사용하는 에이전트를 만듭니다.
    - 사용자의 프롬프트 메시지를 사용하여 스레드를 실행합니다.
    - 오류가 발생할 경우 실행 상태를 확인합니다.
    - 완료된 스레드에서 메시지를 검색하고 에이전트가 보낸 마지막 메시지를 표시합니다.
    - 대화 내용을 표시합니다.
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
    
    애플리케이션은 인증된 Azure 세션에 대한 자격 증명을 사용하여 실행하여 프로젝트에 연결하고 에이전트를 만들고 실행합니다.

1. 메시지가 표시되면 다음과 같은 프롬프트를 입력합니다.

    ```
   I have a technical problem
    ```

    > **팁**: 속도 제한을 초과하여 앱이 실패하는 경우. 몇 초 정도 기다렸다가 다시 시도하세요. 구독에서 사용할 수 있는 할당량이 부족한 경우 모델이 응답하지 않을 수 있습니다.

1. 응답을 봅니다. 에이전트가 이메일 주소와 문제에 대한 설명을 요청할 수 있습니다. 이메일 주소(예: `alex@contoso.com`)와 문제 설명(예: `my computer won't start`)은 무엇이든 사용할 수 있습니다.

    충분한 정보가 있는 경우 에이전트는 필요에 따라 함수를 사용하도록 선택해야 합니다.

1. 원하는 경우 대화를 계속할 수 있습니다. 스레드는 *상태 저장*이므로 대화 내용을 유지합니다. 즉, 에이전트에 각 응답에 대한 전체 컨텍스트가 있습니다. 완료되면 `quit`을(를) 입력합니다.
1. 스레드에서 검색된 대화 메시지와 생성된 티켓을 검토합니다.
1. 이 도구는 앱 폴더에 지원 티켓을 저장했어야 합니다. `ls` 명령을 사용하여 확인한 다음 `cat` 명령을 사용하여 다음과 같이 파일 내용을 볼 수 있습니다.

    ```
   cat ticket-<ticket_num>.txt
    ```

## 정리

연습을 마쳤으므로 불필요한 리소스 사용을 방지하기 위해 만든 클라우드 리소스를 삭제해야 합니다.

1. `https://portal.azure.com`에서 [Azure Portal](https://portal.azure.com)을 열고 이 연습에 사용된 허브 리소스를 배포한 리소스 그룹의 내용을 확인합니다.
1. 도구 모음에서 **리소스 그룹 삭제**를 선택합니다.
1. 리소스 그룹 이름을 입력하고 삭제할 것인지 확인합니다.