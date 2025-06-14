---
lab:
  title: Azure AI 파운드리를 사용하여 다중 에이전트 솔루션 개발
  description: Azure AI 파운드리 에이전트 서비스를 사용하여 공동 작업을 위한 여러 에이전트 구성 방법 알아보기
---

# 다중 에이전트 솔루션 개발

이 연습에서는 Azure AI 파운드리 에이전트 서비스를 사용하여 여러 AI 에이전트를 오케스트레이션하는 프로젝트를 만듭니다. 이 연습에서는 파티를 던전으로 안내하는 역할을 맡은 퀘스트 마스터 에이전트를 만듭니다. 파티는 전사, 치유사, 정찰병을 대표하는 연결된 AI 에이전트들로 구성되어 있습니다. 퀘스트 마스터 에이전트는 사용자로부터 시나리오를 수신하고 그에 따라 파티 구성원에게 작업을 위임합니다. 그럼 시작하겠습니다.

이 연습을 완료하는 데 약 **30**분 정도 소요됩니다.

## Azure AI 파운드리 프로젝트에서 모델 배포

Azure AI 파운드리 프로젝트에 모델을 배포하는 것부터 시작해 보겠습니다.

1. 웹 브라우저에서 [Azure AI 파운드리 포털](https://ai.azure.com)(`https://ai.azure.com`)을 열고 Azure 자격 증명을 사용하여 로그인합니다. 처음 로그인할 때 열리는 팁이나 빠른 시작 창을 닫고, 필요한 경우 왼쪽 위에 있는 **Azure AI 파운드리** 로고를 사용하여 다음 이미지와 유사한 홈페이지로 이동합니다(**도움말** 창이 열려 있는 경우 닫습니다).

    ![Azure AI Foundry 포털의 스크린샷.](./Media/ai-foundry-home.png)

1. 홈페이지의 **모델 및 기능 탐색** 섹션에서 프로젝트에 사용할 `gpt-4o` 모델을 검색합니다.
1. 검색 결과에서 **GPT-4O** 모델을 선택하여 세부 정보를 확인한 다음 해당 모델의 페이지 상단에서 **이 모델 사용**을 선택합니다.
1. 프로젝트를 만들라는 메시지가 표시되면 프로젝트의 유효한 이름을 입력하고 **고급 옵션**을 펼칩니다.
1. 프로젝트에 대한 다음 설정을 확인합니다.
    - **Azure AI 파운드리 리소스**: *Azure AI 파운드리 리소스의 유효한 이름*
    - **구독**: ‘Azure 구독’
    - **리소스 그룹**: ‘리소스 그룹 만들기 또는 선택’
    - **지역**: **AI 서비스 지원 위치 *선택***\*

    > \* 일부 Azure AI 리소스는 지역 모델 할당량에 의해 제한됩니다. 연습 후반부에 할당량 한도를 초과하는 경우 다른 지역에서 다른 리소스를 만들어야 할 수도 있습니다.

1. **만들기**를 선택하고 선택한 GPT-4 모델 배포를 포함한 프로젝트가 생성될 때까지 기다립니다.
1. 프로젝트를 만들면 채팅 플레이그라운드가 자동으로 열립니다.

    > **참고**: 이 연습에서는 이 모델의 기본 TPM 설정이 너무 낮을 수 있습니다. TPM이 낮을수록 사용 중인 구독에서 사용 가능한 할당량을 과도하게 사용하지 않는 데 도움이 됩니다. 

1. 왼쪽 탐색 창에서 **모델 및 엔드포인트**를 선택하고 **GPT-4o** 배포를 선택합니다.

1. **편집**을 선택한 다음 **분당 토큰 속도 제한**을 높입니다.

   > **참고**: 이 연습에 사용된 데이터는 40,000TPM이면 충분합니다. 사용 가능한 할당량이 이 수치 이하이면 연습을 완료할 수 있지만 속도 제한을 초과하는 경우 기다린 다음 프롬프트를 다시 제출해야 할 수 있습니다.

1. 왼쪽 탐색 창에서 **개요**를 선택하면 다음과 같은 프로젝트의 메인 페이지가 표시됩니다.

    > **참고**: *권한 부족** 오류가 표시되면 **수정** 버튼을 사용하여 문제를 해결합니다.

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
   cd ai-agents/Labfiles/06-build-multi-agent-solution/Python
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

1. 코드 파일에서 **your_project_endpoint** 자리 표시자를 프로젝트의 엔드포인트(Azure AI 파운드리 포털의 프로젝트 **개요** 페이지에서 복사)로 바꾸고 **your_model_deployment** 자리 표시자를 GPT-4o 모델 배포에 할당한 이름으로 바꿉니다.

1. 자리 표시자를 바꾼 후 **Ctrl+S** 명령을 사용하여 변경 내용을 저장한 다음 **Ctrl+Q** 명령을 사용하여 Cloud Shell 명령줄을 열어 두고 코드 편집기를 닫습니다.

### AI 에이전트 만들기

이제 다중 에이전트 솔루션을 위한 에이전트를 만들 준비가 되었습니다. 그럼 시작하겠습니다.

1. 다음 명령을 입력하여 **agent_quest.py** 파일을 편집합니다.

    ```
   code agent_quest.py
    ```

1. 파일의 코드를 검토하면서, 각 에이전트의 이름과 지침에 해당하는 문자열이 포함되어 있음을 확인합니다.

1. **Add references** 주석을 찾아 다음 코드를 추가하여 필요한 클래스를 가져옵니다.

    ```python
    # Add references
    from azure.ai.agents import AgentsClient
    from azure.ai.agents.models import ConnectedAgentTool, MessageRole, ListSortOrder
    from azure.identity import DefaultAzureCredential
    ```

1. **Create the healer agent on the Azure AI agent service**라는 주석을 찾아 다음 코드를 추가하여 Azure AI 에이전트를 만듭니다.

    ```python
    # Create the healer agent on the Azure AI agent service
    healer_agent = agents_client.create_agent(
        model=model_deployment,
        name=healer_agent_name,
        instructions=healer_instructions
    )
    ```

    이 코드는 Azure AI 에이전트 클라이언트에서 에이전트 정의를 만듭니다.

1. **Create a connected agent tool for the healer agent**라는 주석을 찾아 다음 코드를 추가합니다.

    ```python
    # Create a connected agent tool for the healer agent
    healer_agent_tool = ConnectedAgentTool(
        id=healer_agent.id, 
        name=healer_agent_name, 
        description="Responsible for healing party members and addressing injuries."
    )
    ```

    이제 다른 파티 멤버 에이전트를 만들어 보겠습니다.

1. **Create the scout agent and connected tool**라는 주석 아래에 다음 코드를 추가합니다.
    
    ```python
    # Create the scout agent and connected tool
    scout_agent = agents_client.create_agent(
        model=model_deployment,
        name=scout_agent_name,
        instructions=scout_instructions
    )
    scout_agent_tool = ConnectedAgentTool(
        id=scout_agent.id, 
        name=scout_agent_name, 
        description="Goes ahead of the main party to perform reconnaissance."
    )
    ```

1. **Create the warrior agent and connected tool**라는 주석 아래에 다음 코드를 추가합니다.
    
    ```python
    # Create the warrior agent and connected tool
    warrior_agent = agents_client.create_agent(
        model=model_deployment,
        name=warrior_agent_name,
        instructions=warrior_instructions
    )
    warrior_agent_tool = ConnectedAgentTool(
        id=warrior_agent.id, 
        name=warrior_agent_name, 
        description="Responds to combat or physical challenges."
    )
    ```


1. **Create a main agent with the Connected Agent tools**라는 주석 아래에 다음 코드를 추가합니다.
    
    ```python
    # Create a main agent with the Connected Agent tools
    agent = agents_client.create_agent(
        model=model_deployment,
        name="quest_master",
        instructions="""
            You are the Questmaster, the intelligent guide of a three-member adventuring party exploring a short dungeon. 
            Based on the scenario, delegate tasks to the appropriate party member. The current party members are: Warrior, Scout, Healer.
            Only include the party member's response, do not provide an analysis or summary.
        """,
        tools=[
            healer_agent_tool.definitions[0],
            scout_agent_tool.definitions[0],
            warrior_agent_tool.definitions[0]
        ]
    )
    ```

1. **Create thread for the chat session**이라는 주석을 찾아 다음 코드를 추가합니다.
    
    ```python
    # Create thread for the chat session
    print("Creating agent thread.")
    thread = agents_client.threads.create()
    ```


1. **Create the quest prompt**라는 주석 아래에 다음 코드를 추가합니다.
    
    ```python
    prompt = "We find a locked door with strange symbols, and the warrior is limping."
    ```

1. **Send a prompt to the agent**라는 주석 아래에 다음 코드를 추가합니다.
    
    ```python
    # Send a prompt to the agent
    message = agents_client.messages.create(
        thread_id=thread.id,
        role=MessageRole.USER,
        content=prompt,
    )
    ```

1. **Create and process Agent run in thread with tools**라는 주석 아래에 다음 코드를 추가합니다.
    
    ```python
    # Create and process Agent run in thread with tools
    print("Processing agent thread. Please wait.")
    run = agents_client.runs.create_and_process(thread_id=thread.id, agent_id=agent.id)
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
   python agent_quest.py
    ```

    다음과 비슷한 결과가 나타나야 합니다.

    ```output
    Creating agent thread.
    Processing agent thread. Please wait.

    MessageRole.USER:
    We find a locked door with strange symbols, and the warrior is limping.

    MessageRole.AGENT:
    - **Scout:** Decipher the celestial patterns of the strange symbols and determine the sequence to unlock the door. 
    - **Healer:** The warrior's injury has been addressed; moderate strain is relieved through healing magic and restorative salve.
    - **Warrior:** Recovered and ready to assist physically or guard the party as we proceed.

    Cleaning up agents:
    Deleted quest master agent.
    Deleted healer agent.
    Deleted scout agent.
    Deleted warrior agent.
    ```

    다른 시나리오를 사용하여 프롬프트를 수정하면 에이전트가 공동 작업하는 방법을 확인할 수 있습니다.

## 정리

Azure AI 에이전트 서비스 탐색을 마친 경우 이 연습에서 만든 리소스를 삭제하여 불필요한 Azure 비용이 발생하지 않도록 해야 합니다.

1. Azure Portal이 포함된 브라우저 탭으로 돌아가서(또는 새 브라우저 탭의 `https://portal.azure.com`에서 [Azure Portal](https://portal.azure.com)을 다시 열고) 이 연습에 사용된 리소스를 배포한 리소스 그룹의 콘텐츠를 확인합니다.

1. 도구 모음에서 **리소스 그룹 삭제**를 선택합니다.

1. 리소스 그룹 이름을 입력하고 삭제할 것인지 확인합니다.
