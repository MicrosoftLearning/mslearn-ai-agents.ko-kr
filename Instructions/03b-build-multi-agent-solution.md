---
lab:
  title: Azure AI 파운드리를 사용하여 다중 에이전트 솔루션 개발
  description: Azure AI 파운드리 에이전트 서비스를 사용하여 공동 작업을 위한 여러 에이전트 구성 방법 알아보기
---

# 다중 에이전트 솔루션 개발

이 연습에서는 Azure AI 파운드리 에이전트 서비스를 사용하여 여러 AI 에이전트를 오케스트레이션하는 프로젝트를 만듭니다. 티켓 심사를 지원하는 AI 솔루션을 디자인합니다. 연결된 에이전트는 티켓의 우선 순위를 평가하고, 팀 할당을 제안하고, 티켓을 완료하는 데 필요한 작업 수준을 결정합니다. 그럼 시작하겠습니다.

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

## AI 에이전트 클라이언트 앱 만들기

이제 에이전트 및 지침을 정의하는 클라이언트 앱을 만들 준비가 되었습니다. 일부 코드는 GitHub 리포지토리에 제공됩니다.

### 환경 준비

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

    > **팁**: Cloud Shell에 명령을 입력하면 출력이 화면 버퍼의 많은 부분을 차지하여 현재 줄의 커서가 가려질 수 있습니다. `cls` 명령을 입력해 화면을 지우면 각 작업에 더 집중할 수 있습니다.

1. 리포지토리가 복제된 경우 다음 명령을 입력하여 작업 디렉터리를 코드 파일이 포함된 폴더로 변경하고 파일을 모두 나열합니다.

    ```
   cd ai-agents/Labfiles/03b-build-multi-agent-solution/Python
   ls -a -l
    ```

    제공된 파일에는 애플리케이션 코드와 구성 설정에 대한 파일이 포함됩니다.

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

1. 코드 파일에서 **your_project_endpoint** 자리 표시자를 프로젝트의 엔드포인트(Azure AI Foundry 포털의 프로젝트 **개요** 페이지에서 복사)로 바꾸고 **your_model_deployment** 자리 표시자를 gpt-4o 모델 배포에 할당한 이름(기본값으로 `gpt-4o`)으로 바꿉니다.

1. 자리 표시자를 바꾼 후 **Ctrl+S** 명령을 사용하여 변경 내용을 저장한 다음 **Ctrl+Q** 명령을 사용하여 Cloud Shell 명령줄을 열어 두고 코드 편집기를 닫습니다.

### AI 에이전트 만들기

이제 다중 에이전트 솔루션을 위한 에이전트를 만들 준비가 되었습니다. 그럼 시작하겠습니다.

1. 다음 명령을 입력하여 **agent_triage.py** 파일을 편집합니다.

    ```
   code agent_triage.py
    ```

1. 파일의 코드를 검토하면서, 각 에이전트의 이름과 지침에 해당하는 문자열이 포함되어 있음을 확인합니다.

1. **Add references** 주석을 찾아 다음 코드를 추가하여 필요한 클래스를 가져옵니다.

    ```python
   # Add references
   from azure.ai.agents import AgentsClient
   from azure.ai.agents.models import ConnectedAgentTool, MessageRole, ListSortOrder, ToolSet, FunctionTool
   from azure.identity import DefaultAzureCredential
    ```

1. 환경 변수에서 프로젝트 엔드포인트 및 모델 이름을 로드하는 코드가 제공되었습니다.

1. **에이전트 클라이언트에 연결하기** 주석을 찾고 다음 코드를 추가하여 프로젝트에 연결된 AgentsClient를 만듭니다.

    ```python
   # Connect to the agents client
   agents_client = AgentsClient(
        endpoint=project_endpoint,
        credential=DefaultAzureCredential(
            exclude_environment_credential=True, 
            exclude_managed_identity_credential=True
        ),
   )
    ```

    이제 AgentsClient를 사용하여 여러 에이전트를 만드는 코드를 추가합니다. 각 에이전트는 지원 티켓을 처리하는 데 있어 특정 역할을 합니다.

    > **팁**: 후속 코드를 추가할 때 `using agents_client:` 문 아래에 올바른 수준의 들여쓰기를 유지해야 합니다.

1. **지원 티켓의 우선 순위를 지정하는 에이전트 만들기** 주석을 찾고 다음 코드를 입력합니다(올바른 수준의 들여쓰기를 유지하도록 주의).

    ```python
   # Create an agent to prioritize support tickets
   priority_agent_name = "priority_agent"
   priority_agent_instructions = """
   Assess how urgent a ticket is based on its description.

   Respond with one of the following levels:
   - High: User-facing or blocking issues
   - Medium: Time-sensitive but not breaking anything
   - Low: Cosmetic or non-urgent tasks

   Only output the urgency level and a very brief explanation.
   """

   priority_agent = agents_client.create_agent(
        model=model_deployment,
        name=priority_agent_name,
        instructions=priority_agent_instructions
   )
    ```

1. **적절한 팀에 티켓을 할당하는 에이전트 만들기** 주석을 찾고 다음 코드를 입력합니다.

    ```python
   # Create an agent to assign tickets to the appropriate team
   team_agent_name = "team_agent"
   team_agent_instructions = """
   Decide which team should own each ticket.

   Choose from the following teams:
   - Frontend
   - Backend
   - Infrastructure
   - Marketing

   Base your answer on the content of the ticket. Respond with the team name and a very brief explanation.
   """

   team_agent = agents_client.create_agent(
        model=model_deployment,
        name=team_agent_name,
        instructions=team_agent_instructions
   )
    ```

1. **지원 티켓의 작업량을 추정하는 에이전트 만들기** 주석을 찾고 다음 코드를 입력합니다.

    ```python
   # Create an agent to estimate effort for a support ticket
   effort_agent_name = "effort_agent"
   effort_agent_instructions = """
   Estimate how much work each ticket will require.

   Use the following scale:
   - Small: Can be completed in a day
   - Medium: 2-3 days of work
   - Large: Multi-day or cross-team effort

   Base your estimate on the complexity implied by the ticket. Respond with the effort level and a brief justification.
   """

   effort_agent = agents_client.create_agent(
        model=model_deployment,
        name=effort_agent_name,
        instructions=effort_agent_instructions
   )
    ```

    지금까지 지원 티켓을 심사하는 데 있어 각각 고유의 역할을 가진 세 개의 에이전트를 만들었습니다. 이제 다른 에이전트에서 사용할 수 있도록 각 에이전트에 대해 ConnectedAgentTool 개체를 만들어 보겠습니다.

1. **지원 에이전트를 위한 연결된 에이전트 도구 만들기** 주석을 찾고 다음 코드를 입력합니다.

    ```python
   # Create connected agent tools for the support agents
   priority_agent_tool = ConnectedAgentTool(
        id=priority_agent.id, 
        name=priority_agent_name, 
        description="Assess the priority of a ticket"
   )
    
   team_agent_tool = ConnectedAgentTool(
        id=team_agent.id, 
        name=team_agent_name, 
        description="Determines which team should take the ticket"
   )
    
   effort_agent_tool = ConnectedAgentTool(
        id=effort_agent.id, 
        name=effort_agent_name, 
        description="Determines the effort required to complete the ticket"
   )
    ```

    이제 필요에 따라 연결된 에이전트를 사용하여 티켓 심사 프로세스를 조정하는 기본 에이전트를 만들 준비가 되었습니다.

1. **연결된 에이전트를 사용하여 지원 티켓 처리를 심사하는 에이전트 만들기** 주석을 찾고 다음 코드를 입력합니다.

    ```python
   # Create an agent to triage support ticket processing by using connected agents
   triage_agent_name = "triage-agent"
   triage_agent_instructions = """
   Triage the given ticket. Use the connected tools to determine the ticket's priority, 
   which team it should be assigned to, and how much effort it may take.
   """

   triage_agent = agents_client.create_agent(
        model=model_deployment,
        name=triage_agent_name,
        instructions=triage_agent_instructions,
        tools=[
            priority_agent_tool.definitions[0],
            team_agent_tool.definitions[0],
            effort_agent_tool.definitions[0]
        ]
   )
    ```

    이제 기본 에이전트를 정의했으므로 프롬프트를 제출하고 다른 에이전트를 사용하여 지원 문제를 심사하도록 할 수 있습니다.

1. **에이전트를 사용하여 지원 문제를 심사하기** 주석을 찾고 다음 코드를 입력합니다.

    ```python
   # Use the agents to triage a support issue
   print("Creating agent thread.")
   thread = agents_client.threads.create()  

   # Create the ticket prompt
   prompt = input("\nWhat's the support problem you need to resolve?: ")
    
   # Send a prompt to the agent
   message = agents_client.messages.create(
        thread_id=thread.id,
        role=MessageRole.USER,
        content=prompt,
   )   
    
   # Run the thread usng the primary agent
   print("\nProcessing agent thread. Please wait.")
   run = agents_client.runs.create_and_process(thread_id=thread.id, agent_id=triage_agent.id)
        
   if run.status == "failed":
        print(f"Run failed: {run.last_error}")

   # Fetch and display messages
   messages = agents_client.messages.list(thread_id=thread.id, order=ListSortOrder.ASCENDING)
   for message in messages:
        if message.text_messages:
            last_msg = message.text_messages[-1]
            print(f"{message.role}:\n{last_msg.text.value}\n")
   
    ```

1. **정리하기** 주석을 찾고 다음 코드를 입력하여 더 이상 필요하지 않은 에이전트를 삭제합니다.

    ```python
   # Clean up
   print("Cleaning up agents:")
   agents_client.delete_agent(triage_agent.id)
   print("Deleted triage agent.")
   agents_client.delete_agent(priority_agent.id)
   print("Deleted priority agent.")
   agents_client.delete_agent(team_agent.id)
   print("Deleted team agent.")
   agents_client.delete_agent(effort_agent.id)
   print("Deleted effort agent.")
    ```
    

1. **CTRL+S** 명령을 사용하여 변경 내용을 코드 파일에 저장합니다. 파일을 열어 두거나(오류를 수정하기 위해 코드를 편집해야 하는 경우) **Ctrl+Q** 명령을 사용하여 Cloud Shell 명령줄을 열어 둔 채 코드 편집기를 닫습니다.

### Azure에 로그인하고 앱 실행

이제 코드를 실행하고 AI 에이전트가 공동 작업하는 모습을 볼 준비가 되었습니다.

1. Cloud Shell 명령줄 창에서 다음 명령을 입력하여 Azure에 로그인합니다.

    ```
   az login
    ```

    **<font color="red">Cloud Shell 세션이 이미 인증되었더라도 Azure에 로그인해야 합니다.</font>**

    > **참고**: 대부분의 시나리오에서는 *az login*을 사용하는 것만으로도 충분합니다. 그러나 여러 테넌트에 구독이 있는 경우 *--tenant* 매개 변수를 사용하여 테넌트 지정해야 할 수 있습니다. 자세한 내용은 [Sign into Azure interactively using the Azure CLI](https://learn.microsoft.com/cli/azure/authenticate-azure-cli-interactively)를 참조하세요.

1. 메시지가 표시되면 지침에 따라 새 탭에서 로그인 페이지를 열고 제공된 인증 코드와 Azure 자격 증명을 입력합니다. 그런 다음 명령줄에서 로그인 프로세스를 완료하고 메시지가 표시되면 Azure AI 파운드리 허브가 포함된 구독을 선택합니다.

1. 로그인한 후 다음 명령을 입력하여 애플리케이션을 실행합니다.

    ```
   python agent_triage.py
    ```

1. `Users can't reset their password from the mobile app.`(와)과 같은 프롬프트를 입력합니다.

    에이전트가 프롬프트를 처리한 후 다음과 같은 값이 출력됩니다.

    ```output
    Creating agent thread.
    Processing agent thread. Please wait.

    MessageRole.USER:
    Users can't reset their password from the mobile app.

    MessageRole.AGENT:
    ### Ticket Assessment

    - **Priority:** High — This issue blocks users from resetting their passwords, limiting access to their accounts.
    - **Assigned Team:** Frontend Team — The problem lies in the mobile app's user interface or functionality.
    - **Effort Required:** Medium — Resolving this problem involves identifying the root cause, potentially updating the mobile app functionality, reviewing API/backend integration, and testing to ensure compatibility across Android/iOS platforms.

    Cleaning up agents:
    Deleted triage agent.
    Deleted priority agent.
    Deleted team agent.
    Deleted effort agent.
    ```

    다른 티켓 시나리오를 사용하여 프롬프트를 수정하면 에이전트가 공동 작업하는 방법을 확인할 수 있습니다. 예로 "Investigate occasional 502 errors from the search endpoint."를 들 수 있습니다.

## 정리

Azure AI 에이전트 서비스 탐색을 마친 경우 이 연습에서 만든 리소스를 삭제하여 불필요한 Azure 비용이 발생하지 않도록 해야 합니다.

1. Azure Portal이 포함된 브라우저 탭으로 돌아가서(또는 새 브라우저 탭의 `https://portal.azure.com`에서 [Azure Portal](https://portal.azure.com)을 다시 열고) 이 연습에 사용된 리소스를 배포한 리소스 그룹의 콘텐츠를 확인합니다.

1. 도구 모음에서 **리소스 그룹 삭제**를 선택합니다.

1. 리소스 그룹 이름을 입력하고 삭제할 것인지 확인합니다.
