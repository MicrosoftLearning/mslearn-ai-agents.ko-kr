---
lab:
  title: 원격 MCP 서버에 AI 에이전트 연결
  description: AI 에이전트와 모델 컨텍스트 프로토콜 도구를 통합하는 방법 알아보기
---

# MCP(모델 컨텍스트 프로토콜)를 사용하여 도구에 AI 에이전트 연결

이 연습에서는 클라우드에 호스트된 MCP 서버에 연결하는 에이전트를 구축해 보겠습니다. 이 에이전트는 AI 기반 검색을 사용하여 개발자가 Microsoft 공식 문서에서 정확한 실시간 답변을 찾을 수 있도록 도와줍니다. 이 기능은 Azure, .NET, Microsoft 365 등 도구의 최신 지침을 사용하여 개발자를 지원하는 도우미를 빌드하는 데 유용합니다. 에이전트는 사용 가능한 MCP 도구를 사용하여 문서를 쿼리하고 관련 결과를 반환합니다.

> **팁**: 이 연습에서 사용된 코드는 Azure AI 에이전트 서비스 MCP 지원 샘플 리포지토리를 기반으로 합니다. 자세한 내용은 [Azure OpenAI 데모](https://github.com/retkowsky/Azure-OpenAI-demos/blob/main/Azure%20Agent%20Service/9%20Azure%20AI%20Agent%20service%20-%20MCP%20support.ipynb)를 참조하거나 [모델 컨텍스트 프로토콜 서버에 연결](https://learn.microsoft.com/en-us/azure/ai-foundry/agents/how-to/tools/model-context-protocol)을 방문하세요.

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
    - **지역**: *지원되는 다음 위치 중에서 선택합니다.* \* 
      * 미국 서부 2
      * 미국 서부
      * 노르웨이 동부
      * 스위스 북부
      * 아랍에미리트 북부
      * 인도 남부

    > \* 일부 Azure AI 리소스는 지역 모델 할당량에 의해 제한됩니다. 연습 후반부에 할당량 한도를 초과하는 경우 다른 지역에서 다른 리소스를 만들어야 할 수도 있습니다.

1. **만들기**를 선택한 다음, 프로젝트가 만들어질 때까지 기다립니다.
1. 메시지가 표시되면 할당량 가용성에 따라 *전역 표준* 또는 *표준* 배포 옵션을 사용하여 **gpt-4o** 모델을 배포합니다.

    >**참고**: 할당량을 사용할 수 있는 경우 에이전트 및 프로젝트를 만들 때 GPT-4o 기본 모델이 자동으로 배포될 수 있습니다.

1. 프로젝트를 만들면 에이전트 플레이그라운드가 열립니다.

1. 왼쪽 탐색 창에서 **개요**를 선택하면 다음과 같은 프로젝트의 메인 페이지가 표시됩니다.

    ![Azure AI 파운드리 프로젝트 개요 페이지의 스크린샷.](./Media/ai-foundry-project.png)

1. **Azure AI Foundry 프로젝트 엔드포인트** 값을 복사합니다. 이 엔드포인트를 사용하여 클라이언트 응용 프로그램에서 프로젝트에 연결할 수 있습니다.

## MCP 함수 도구를 사용하는 에이전트 개발

AI 파운드리에서 프로젝트를 만들었으므로 이제 AI 에이전트를 MCP 서버와 통합하는 앱을 개발해 보겠습니다.

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
   cd ai-agents/Labfiles/03c-use-agent-tools-with-mcp/Python
   ls -a -l
    ```

### 애플리케이션 설정 구성

1. Cloud Shell 명령줄 창에서 다음 명령을 입력하여 사용할 라이브러리를 설치합니다.

    ```
   python -m venv labenv
   ./labenv/bin/Activate.ps1
   pip install -r requirements.txt --pre azure-ai-projects mcp
    ```

    >**참고:** 라이브러리 설치 중에 표시되는 경고 또는 오류 메시지를 무시할 수 있습니다.

1. 제공된 구성 파일을 편집하려면 다음 명령을 입력합니다.

    ```
   code .env
    ```

    코드 편집기에서 파일이 열립니다.

1. 코드 파일에서 **your_project_endpoint** 자리 표시자를 프로젝트의 엔드포인트(Azure AI Foundry 포털의 프로젝트 **개요** 페이지에서 복사됨)로 바꾸고 MODEL_DEPLOYMENT_NAME 변수가 모델 배포 이름(*gpt-4o*)으로 설정되어 있는지 확인합니다.

1. 자리 표시자를 바꾼 후에는 **CTRL+S** 명령을 사용하여 변경 내용을 저장한 다음 **CTRL+Q** 명령을 사용하여 Cloud Shell 명령줄을 열어둔 상태에서 코드 편집기를 닫습니다.

### 원격 MCP 서버에 Azure AI 에이전트 연결

이 작업을 통해 원격 MCP 서버에 연결하고, AI 에이전트를 준비하고, 사용자 프롬프트를 실행하게 됩니다.

1. 제공된 코드 파일을 편집하려면 다음 명령을 입력합니다.

    ```
   code client.py
    ```

    코드 편집기에서 파일이 열립니다.

1. **Add references** 주석을 찾고 다음 코드를 추가하여 클래스를 가져옵니다.

    ```python
   # Add references
   from azure.identity import DefaultAzureCredential
   from azure.ai.agents import AgentsClient
   from azure.ai.agents.models import McpTool, ToolSet, ListSortOrder
    ```

1. **Connect to the agents client** 주석을 찾아 다음 코드를 추가하여 현재 Azure 자격 증명을 사용하여 Azure AI 프로젝트에 연결합니다.

    ```python
   # Connect to the agents client
   agents_client = AgentsClient(
        endpoint=project_endpoint,
        credential=DefaultAzureCredential(
            exclude_environment_credential=True,
            exclude_managed_identity_credential=True
        )
   )
    ```

1. **에이전트 MCP 도구 초기화** 주석 아래에 다음 코드를 추가하세요.

    ```python
   # Initialize agent MCP tool
   mcp_tool = McpTool(
        server_label=mcp_server_label,
        server_url=mcp_server_url,
   )
    
   mcp_tool.set_approval_mode("never")
    
   toolset = ToolSet()
   toolset.add(mcp_tool)
    ```

    이 코드는 Microsft Learn Docs 원격 MCP 서버에 연결됩니다. 이는 클라이언트가 Microsoft 공식 문서에서 직접 신뢰할 수 있고 최신 정보에 액세스할 수 있도록 하는 클라우드에 호스트된 서비스입니다.

1. **새 에이전트 만들기** 주석을 찾고 다음 코드를 추가합니다.

    ```python
   # Create a new agent
   agent = agents_client.create_agent(
        model=model_deployment,
        name="my-mcp-agent",
        instructions="""
        You have access to an MCP server called `microsoft.docs.mcp` - this tool allows you to 
        search through Microsoft's latest official documentation. Use the available MCP tools 
        to answer questions and perform tasks."""
   )
    ```

    이 코드에서는 에이전트의 지침과 MCO 도구 정의를 제공합니다.

1. **커뮤니케이션을 위한 스레드 만들기**라는 주석을 찾아 다음 코드를 추가합니다.

    ```python
   # Create thread for communication
   thread = agents_client.threads.create()
   print(f"Created thread, ID: {thread.id}")
    ```

1. **스레드에 메시지 작성**이라는 주석을 찾아 다음 코드를 추가합니다.

    ```python
   # Create a message on the thread
   prompt = input("\nHow can I help?: ")
   message = agents_client.messages.create(
        thread_id=thread.id,
        role="user",
        content=prompt,
   )
   print(f"Created message, ID: {message.id}")
    ```

1. **승인 모드 설정** 주석을 찾아 다음 코드를 추가합니다.

    ```python
    # Set approval mode
    mcp_tool.set_approval_mode("never")
    ```

    이렇게 하면 에이전트가 사용자 승인 없이 MCP 도구를 자동으로 호출할 수 있습니다. 승인을 요구하려면 `mcp_tool.update_headers`를 사용하여 헤더 값을 제공해야 합니다.

1. **MCP 도구를 사용하여 스레드에서 실행되는 에이전트 만들기 및 처리** 주석을 찾고 다음 코드를 추가합니다.

    ```python
   # Create and process agent run in thread with MCP tools
   run = agents_client.runs.create_and_process(thread_id=thread.id, agent_id=agent.id, toolset=toolset)
   print(f"Created run, ID: {run.id}")
    ```
    
    AI 에이전트는 연결된 MCP 도구를 자동으로 호출하여 즉각적인 요청을 처리할 수 있습니다. 이 프로세스를 설명하기 위해 **실행 단계 및 도구 호출 표시** 주석 아래에 제공된 코드는 MCP 서버에서 호출된 모든 도구를 출력하게 됩니다.

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
   python client.py
    ```

1. 메시지가 표시되면 다음과 같은 기술 정보에 대한 요청을 입력합니다.

    ```
    Give me the Azure CLI commands to create an Azure Container App with a managed identity.
    ```

1. 에이전트가 MCP 서버를 사용하여 요청된 정보를 검색하는 데 적합한 도구를 찾아 프롬프트를 처리할 때까지 기다립니다. 다음과 비슷한 결과가 나타나야 합니다.

    ```
    Created agent, ID: <<agent-id>>
    MCP Server: mslearn at https://learn.microsoft.com/api/mcp
    Created thread, ID: <<thread-id>>
    Created message, ID: <<message-id>>
    Created run, ID: <<run-id>>
    Run completed with status: RunStatus.COMPLETED
    Step <<step1-id>> status: completed

    Step <<step2-id>> status: completed
    MCP Tool calls:
        Tool Call ID: <<tool-call-id>>
        Type: mcp
        Type: microsoft_code_sample_search


    Conversation:
    --------------------------------------------------
    ASSISTANT: You can use Azure CLI to create an Azure Container App with a managed identity (either system-assigned or user-assigned). Below are the relevant commands and workflow:

    ---

    ### **1. Create a Resource Group**
    '''azurecli
    az group create --name myResourceGroup --location eastus
    '''
    

    {{continued...}}

    By following these steps, you can deploy an Azure Container App with either system-assigned or user-assigned managed identities to integrate seamlessly with other Azure services.
    --------------------------------------------------
    USER: Give me the Azure CLI commands to create an Azure Container App with a managed identity.
    --------------------------------------------------
    Deleted agent
    ```

    에이전트가 MCP 도구 `microsoft_code_sample_search`를 자동으로 호출하여 요청을 이행할 수 있었음을 확인하세요.

1. `python client.py` 명령을 사용해 앱을 다시 실행하여 다른 정보를 요청할 수 있습니다. 에이전트는 매번 MCP 도구를 사용하여 기술 설명서 찾기를 시도합니다.

## 정리

연습을 마쳤으므로 불필요한 리소스 사용을 방지하기 위해 만든 클라우드 리소스를 삭제해야 합니다.

1. [Azure Portal](https://portal.azure.com)을 `https://portal.azure.com`에서 열고 이 연습에서 사용한 허브 리소스를 배포한 리소스 그룹의 내용을 확인합니다.
1. 도구 모음에서 **리소스 그룹 삭제**를 선택합니다.
1. 리소스 그룹 이름을 입력하고 삭제할 것인지 확인합니다.
