---
lab:
  title: 다중 에이전트 솔루션 개발
  description: 의미 체계 커널 SDK를 사용하여 여러 에이전트가 공동 작업하도록 구성하는 방법을 알아봅니다.
---

# 다중 에이전트 솔루션 개발

이 연습에서는 의미 체계 커널 SDK를 사용하여 두 개의 AI 에이전트를 오케스트레이션하는 프로젝트를 만듭니다. *인시던트 관리자* 에이전트는 문제에 대한 서비스 로그 파일을 분석합니다. 문제가 발견되면 인시던트 관리자가 해결 작업을 권장하고 *DevOps 도우미* 에이전트는 권장 사항을 수신하고 수정 기능을 호출하고 해결 작업을 수행합니다. 그러면 인시던트 관리자 에이전트가 업데이트된 로그를 검토하여 해결에 성공했는지 확인합니다.

이 연습에서는 4개의 샘플 로그 파일이 제공됩니다. DevOps 도우미 에이전트 코드는 몇 가지 예제 로그 메시지로 샘플 로그 파일만 업데이트합니다.

이 연습을 완료하는 데 약 **30**분 정도 소요됩니다.

> **참고**: 이 연습에 사용된 일부 기술은 미리 보기이거나 현재 개발 중에 있습니다. 예기치 않은 동작, 경고 또는 오류가 발생할 수 있습니다.

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

1. **설정** 창에서 모델 배포의 이름을 기록합니다(**GPT-4o**이어야 함). **모델 및 엔드포인트** 페이지에서 배포를 확인하면 이를 확인할 수 있습니다(왼쪽 탐색 창에서 해당 페이지를 열면 됩니다).
1. 왼쪽 탐색 창에서 **개요**를 선택하면 다음과 같은 프로젝트의 메인 페이지가 표시됩니다.

    > **참고**: *권한 부족** 오류가 표시되면 **수정** 버튼을 사용하여 문제를 해결합니다.

    ![Azure AI 파운드리 포털의 Azure AI 프로젝트 세부 정보 스크린샷.](./Media/ai-foundry-project.png)

## AI 에이전트 클라이언트 앱 만들기

이제 에이전트와 사용자 지정 함수를 정의하는 클라이언트 앱을 만들 준비가 되었습니다. 일부 코드는 GitHub 리포지토리에 제공됩니다.

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
   cd ai-agents/Labfiles/05-agent-orchestration/Python
   ls -a -l
    ```

    제공된 파일에는 애플리케이션 코드와 구성 설정에 대한 파일이 포함됩니다.

### 애플리케이션 설정 구성

1. Cloud Shell 명령줄 창에서 다음 명령을 입력하여 사용할 라이브러리를 설치합니다.

    ```
   python -m venv labenv
   ./labenv/bin/Activate.ps1
   pip install python-dotenv azure-identity semantic-kernel[azure] 
    ```

    > **참고**: *semantic-kernel[azure]* 을 설치하면 *azure-ai-projects*의 Semantic Kernel과 호환되는 버전이 자동으로 설치됩니다.

1. 제공된 구성 파일을 편집하려면 다음 명령을 입력합니다.

    ```
   code .env
    ```

    코드 편집기에서 파일이 열립니다.

1. 코드 파일에서 **your_project_endpoint** 자리 표시자를 프로젝트의 엔드포인트(Azure AI 파운드리 포털의 프로젝트 **개요** 페이지에서 복사)로 바꾸고 **your_model_deployment** 자리 표시자를 GPT-4o 모델 배포에 할당한 이름으로 바꿉니다.

1. 자리 표시자를 바꾼 후 **Ctrl+S** 명령을 사용하여 변경 내용을 저장한 다음 **Ctrl+Q** 명령을 사용하여 Cloud Shell 명령줄을 열어 두고 코드 편집기를 닫습니다.

### AI 에이전트 만들기

이제 다중 에이전트 솔루션을 위한 에이전트를 만들 준비가 되었습니다. 그럼 시작하겠습니다.

1. 다음 명령을 입력하여 **agent_chat.py** 파일을 편집합니다.

    ```
   code agent_chat.py
    ```

1. 파일의 코드를 검토하여 다음을 포함하고 있는지 확인합니다.
    - 두 에이전트의 이름과 지침을 정의하는 상수.
    - 다중 에이전트 솔루션을 구현하는 대부분의 코드가 추가될 **기본** 함수.
    - 대화에서 각 턴마다 어떤 에이전트를 선택할지 결정하는 로직을 구현하는 데 사용할 **SelectionStrategy** 클래스.
    - 대화를 언제 종료할지를 결정하는 로직을 구현하는 데 사용할 **ApprovalTerminationStrategy** 클래스.
    - DevOps 작업을 수행하는 함수를 포함하는 **DevopsPlugin** 클래스.
    - 로그 파일을 읽고 쓰는 함수가 포함된 **LogFilePlugin** 클래스.

    먼저 *인시던트 관리자* 에이전트를 만듭니다. 이 에이전트는 서비스 로그 파일을 분석하고, 잠재적인 문제를 식별하고, 해결 작업을 권장하거나 필요한 경우 문제를 에스컬레이션합니다.

1. **INCIDENT_MANAGER_INSTRUCTIONS** 문자열을 확인합니다. 이는 에이전트에 대한 지침입니다.

1. **기본** 함수에서 **Create the incident manager agent on the Azure AI agent service**라는 주석을 찾고 다음 코드를 추가하여 Azure AI 에이전트를 만듭니다.

    ```python
   # Create the incident manager agent on the Azure AI agent service
   incident_agent_definition = await client.agents.create_agent(
        model=ai_agent_settings.model_deployment_name,
        name=INCIDENT_MANAGER,
        instructions=INCIDENT_MANAGER_INSTRUCTIONS
   )
    ```

    이 코드는 Azure AI Project 클라이언트에 에이전트 정의를 만듭니다.

1. **Create a Semantic Kernel agent for the Azure AI incident manager agent**라는 주석을 찾고, Azure AI 에이전트 정의를 기반으로 의미 체계 커널 에이전트를 만드는 다음 코드를 추가합니다.

    ```python
   # Create a Semantic Kernel agent for the Azure AI incident manager agent
   agent_incident = AzureAIAgent(
        client=client,
        definition=incident_agent_definition,
        plugins=[LogFilePlugin()]
   )
    ```

    이 코드는 **LogFilePlugin**에 대한 액세스 권한이 있는 의미 체계 커널 에이전트를 만듭니다. 이 플러그인을 사용하면 에이전트가 로그 파일 내용을 읽을 수 있습니다.

    이제 두 번째 에이전트를 만들어 보겠습니다. 이 에이전트는 문제에 응답하고 이를 해결하기 위해 DevOps 작업을 수행합니다.

1. 코드 파일 맨 위에서 **DEVOPS_ASSISTANT_INSTRUCTIONS** 문자열을 잠시 확인합니다. 다음은 새 DevOps 도우미 에이전트에 제공할 지침입니다.

1. **Create the devops agent on the Azure AI agent service**라는 주석을 찾아, Azure AI 에이전트 정의를 만드는 다음 코드를 추가합니다.
    
    ```python
   # Create the devops agent on the Azure AI agent service
   devops_agent_definition = await client.agents.create_agent(
        model=ai_agent_settings.model_deployment_name,
        name=DEVOPS_ASSISTANT,
        instructions=DEVOPS_ASSISTANT_INSTRUCTIONS,
   )
    ```

1. **Create a Semantic Kernel agent for the devops Azure AI agent**라는 주석을 찾아, Azure AI 에이전트 정의를 기반으로 의미 체계 커널 에이전트를 생성하는 다음 코드를 추가합니다.
    
    ```python
   # Create a Semantic Kernel agent for the devops Azure AI agent
   agent_devops = AzureAIAgent(
        client=client,
        definition=devops_agent_definition,
        plugins=[DevopsPlugin()]
   )
    ```

    **DevopsPlugin**을 사용하면 에이전트가 서비스 다시 시작 또는 트랜잭션 롤백과 같은 DevOps 작업을 시뮬레이션할 수 있습니다.

### 그룹 채팅 전략 정의

이제 대화에서 다음 턴을 수행할 에이전트를 결정하고 대화를 종료해야 하는 시점을 결정하는 데 사용되는 로직을 제공해야 합니다.

다음 턴을 수행해야 하는 에이전트를 식별하는 **SelectionStrategy**부터 시작해 보겠습니다.

1. **SelectionStrategy** 클래스(**main** 함수 아래)에서 **Select the next agent that should take the next turn in the chat**라는 주석을 찾아, 선택 함수를 정의하는 다음 코드를 추가합니다.

    ```python
   # Select the next agent that should take the next turn in the chat
   async def select_agent(self, agents, history):
        """"Check which agent should take the next turn in the chat."""

        # The Incident Manager should go after the User or the Devops Assistant
        if (history[-1].name == DEVOPS_ASSISTANT or history[-1].role == AuthorRole.USER):
            agent_name = INCIDENT_MANAGER
            return next((agent for agent in agents if agent.name == agent_name), None)
        
        # Otherwise it is the Devops Assistant's turn
        return next((agent for agent in agents if agent.name == DEVOPS_ASSISTANT), None)
    ```

    이 코드는 매 턴마다 실행되어 어떤 에이전트가 응답할지를 결정하며 채팅 기록을 확인하여 마지막으로 응답한 에이전트를 판단합니다.

    이제 목표가 완료되어 대화가 종료될 때 신호를 보낼 수 있도록 **ApprovalTerminationStrategy** 클래스를 구현해 보겠습니다.

1. **ApprovalTerminationStrategy** 클래스에서 **End the chat if the agent has indicated there is no action needed**라는 주석을 찾고 다음 코드를 추가하여 종료 함수를 정의합니다.

    ```python
   # End the chat if the agent has indicated there is no action needed
   async def should_agent_terminate(self, agent, history):
        """Check if the agent should terminate."""
        return "no action needed" in history[-1].content.lower()
    ```

    커널은 에이전트의 응답 후에 이 함수를 호출하여 완료 조건이 충족되는지 확인합니다. 이 경우 인시던트 관리자가 "No action needed"라고 응답하면 목표가 충족된 것입니다. 이 문구는 인시던트 관리자 에이전트 지침에 정의되어 있습니다.

### 그룹 채팅 구현

이제 두 개의 에이전트와 이들이 번갈아 가며 채팅을 종료하도록 돕는 전략이 있으므로 그룹 채팅을 구현할 수 있습니다.

1. 기본 함수 위쪽으로 돌아가, **Add the agents to a group chat with a custom termination and selection strategy**라는 주석을 찾고 그룹 채팅을 생성하는 다음 코드를 추가합니다.

    ```python
   # Add the agents to a group chat with a custom termination and selection strategy
   chat = AgentGroupChat(
        agents=[agent_incident, agent_devops],
        termination_strategy=ApprovalTerminationStrategy(
            agents=[agent_incident], 
            maximum_iterations=10, 
            automatic_reset=True
        ),
        selection_strategy=SelectionStrategy(agents=[agent_incident,agent_devops]),      
   )
    ```

    이 코드에서는, 인시던트 관리자 및 DevOps 에이전트를 사용하여 에이전트 그룹 채팅 개체를 만듭니다. 또한 채팅에 대한 종료 및 선택 전략을 정의합니다. **ApprovalTerminationStrategy**는 DevOps 에이전트가 아닌 인시던트 관리자 에이전트에만 연결되어 있다는 점에 주의합니다. 이로써 인시던트 관리자 에이전트가 채팅 종료를 알리는 역할을 담당하게 됩니다. **SelectionStrategy**에는 채팅에서 턴을 맡아야 하는 모든 에이전트가 포함됩니다.

    자동 재설정 플래그는 채팅이 종료될 때 자동으로 채팅을 지웁니다. 이렇게 하면 채팅 기록 개체에 너무 많은 불필요한 토큰을 사용하지 않고도 에이전트가 파일을 계속 분석할 수 있습니다. 

1. **Append the current log file to the chat**라는 주석을 찾고 가장 최근에 읽은 로그 파일 텍스트를 채팅에 추가하는 다음 코드를 입력합니다.

    ```python
   # Append the current log file to the chat
   await chat.add_chat_message(logfile_msg)
   print()
    ```

1. **Invoke a response from the agents**라는 주석 을 찾아, 그룹 채팅을 호출하는 다음 코드를 추가합니다.

    ```python
   # Invoke a response from the agents
   async for response in chat.invoke():
        if response is None or not response.name:
            continue
        print(f"{response.content}")
    ```

    채팅을 트리거하는 코드입니다. 로그 파일 텍스트가 메시지로 추가되었으므로 선택 전략은 어떤 에이전트가 이를 읽고 응답해야 하는지 결정하고, 그 이후 종료 전략의 조건이 충족되거나 최대 반복 횟수에 도달할 때까지 에이전트 간에 대화가 계속됩니다.

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
   python agent_chat.py
    ```

    다음과 비슷한 결과가 나타나야 합니다.

    ```output
    
    INCIDENT_MANAGER > /home/.../logs/log1.log | Restart service ServiceX
    DEVOPS_ASSISTANT > Service ServiceX restarted successfully.
    INCIDENT_MANAGER > No action needed.

    INCIDENT_MANAGER > /home/.../logs/log2.log | Rollback transaction for transaction ID 987654.
    DEVOPS_ASSISTANT > Transaction rolled back successfully.
    INCIDENT_MANAGER > No action needed.

    INCIDENT_MANAGER > /home/.../logs/log3.log | Increase quota.
    DEVOPS_ASSISTANT > Successfully increased quota.
    (continued)
    ```

    > **참고**: 앱에는 TPM 속도 제한이 초과될 위험을 줄이기 위해 각 로그 파일을 처리하는 동안 대기하는 코드 및 제한이 초과되는 경우를 대비한 예외 처리가 포함되어 있습니다. 구독에서 사용할 수 있는 할당량이 부족한 경우 모델이 응답하지 않을 수 있습니다.

1. **logs** 폴더의 로그 파일이 DevopsAssistant의 해결 작업 메시지로 업데이트되었는지 확인합니다.

    예를 들어 log1.log에는 다음 로그 메시지가 추가되어야 합니다.

    ```log
    [2025-02-27 12:43:38] ALERT  DevopsAssistant: Multiple failures detected in ServiceX. Restarting service.
    [2025-02-27 12:43:38] INFO  ServiceX: Restart initiated.
    [2025-02-27 12:43:38] INFO  ServiceX: Service restarted successfully.
    ```

## 요약

이 연습에서는 Azure AI 에이전트 서비스 및 의미 체계 커널 SDK를 사용하여 문제를 자동으로 감지하고 해결 방법을 적용할 수 있는 AI 인시던트 및 DevOps 에이전트를 만들었습니다. 잘했습니다!

## 정리

Azure AI 에이전트 서비스 탐색을 마친 경우 이 연습에서 만든 리소스를 삭제하여 불필요한 Azure 비용이 발생하지 않도록 해야 합니다.

1. Azure Portal이 포함된 브라우저 탭으로 돌아가서(또는 새 브라우저 탭의 `https://portal.azure.com`에서 [Azure Portal](https://portal.azure.com)을 다시 열고) 이 연습에 사용된 리소스를 배포한 리소스 그룹의 콘텐츠를 확인합니다.

1. 도구 모음에서 **리소스 그룹 삭제**를 선택합니다.

1. 리소스 그룹 이름을 입력하고 삭제할 것인지 확인합니다.
