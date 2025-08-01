# MCP(모델 컨텍스트 프로토콜)를 사용하여 도구에 AI 에이전트 연결

이 연습에서는 MCP 서버에 연결하고 호출 가능한 함수를 자동으로 검색할 수 있는 에이전트를 만듭니다.

화장품 소매업체를 위한 간단한 재고 평가 에이전트를 빌드하려고 합니다. 에이전트는 MCP 서버를 사용하여 재고에 대한 정보를 검색하고 재입고 또는 재고 정리를 제안할 수 있습니다.

> **팁**: 이 연습에서 사용되는 코드는 Python용 Azure AI Foundry 및 MCP SDK를 기준으로 합니다. Microsoft .NET용 SDK를 사용하여 유사한 솔루션을 개발할 수 있습니다. 자세한 내용은 [Azure AI Foundry SDK 클라이언트 라이브러리](https://learn.microsoft.com/azure/ai-foundry/how-to/develop/sdk-overview) 및 [MCP C# SDK](https://modelcontextprotocol.github.io/csharp-sdk/api/ModelContextProtocol.html)를 참조하세요.

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
1. 메시지가 표시되면 할당량 가용성에 따라 *전역 표준* 또는 *표준* 배포 옵션을 사용하여 **gpt-4o** 모델을 배포합니다.

    >**참고**: 할당량을 사용할 수 있는 경우 에이전트 및 프로젝트를 만들 때 GPT-4o 기본 모델이 자동으로 배포될 수 있습니다.

1. 프로젝트를 만들면 에이전트 플레이그라운드가 열립니다.

1. 왼쪽 탐색 창에서 **개요**를 선택하면 다음과 같은 프로젝트의 메인 페이지가 표시됩니다.

    ![Azure AI 파운드리 프로젝트 개요 페이지의 스크린샷.](./Media/ai-foundry-project.png)

1. 클라이언트 응용 프로그램에서 프로젝트에 연결하는 데 사용하므로 **Azure AI 파운드리 프로젝트 엔드포인트** 값을 메모장에 복사합니다.

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
   cd ai-agents/Labfiles/03d-use-agent-tools-with-mcp/Python
   ls -a -l
    ```

    제공된 파일에는 클라이언트 및 서버 응용 프로그램 코드가 포함됩니다. 모델 컨텍스트 프로토콜은 AI 모델을 다양한 데이터 원본 및 도구에 연결하는 표준화된 방법을 제공합니다. `client.py` 및 `server.py`를 분리하여 에이전트 논리 및 도구 정의를 모듈식으로 유지하고 실제 아키텍처를 시뮬레이트합니다. 
    
    `server.py`는 에이전트가 사용할 수 있는 도구를 정의하여 백 엔드 서비스 또는 비즈니스 논리를 시뮬레이트합니다. 
    `client.py`는 필요할 때 AI 에이전트 설정, 사용자 프롬프트 및 도구 호출을 처리합니다.

### 애플리케이션 설정 구성

1. Cloud Shell 명령줄 창에서 다음 명령을 입력하여 사용할 라이브러리를 설치합니다.

    ```
   python -m venv labenv
   ./labenv/bin/Activate.ps1
   pip install -r requirements.txt azure-ai-projects mcp
    ```

    >**참고:** 라이브러리 설치 중에 표시되는 경고 또는 오류 메시지를 무시할 수 있습니다.

1. 제공된 구성 파일을 편집하려면 다음 명령을 입력합니다.

    ```
   code .env
    ```

    코드 편집기에서 파일이 열립니다.

1. 코드 파일에서 **your_project_endpoint** 자리 표시자를 프로젝트의 엔드포인트(Azure AI Foundry 포털의 프로젝트 **개요** 페이지에서 복사됨)로 바꾸고 MODEL_DEPLOYMENT_NAME 변수가 모델 배포 이름(*gpt-4o*)으로 설정되어 있는지 확인합니다.

1. 자리 표시자를 바꾼 후에는 **CTRL+S** 명령을 사용하여 변경 내용을 저장한 다음 **CTRL+Q** 명령을 사용하여 Cloud Shell 명령줄을 열어둔 상태에서 코드 편집기를 닫습니다.

### MCP 서버 구현

MCP(모델 컨텍스트 프로토콜) 서버는 호출 가능 도구를 호스트하는 구성 요소입니다. 이러한 도구는 AI 에이전트에게 노출될 수 있는 Python 함수입니다. 도구에 `@mcp.tool()` 주석이 추가되면 클라이언트가 검색할 수 있게 되므로 AI 에이전트가 대화 또는 작업 도중 해당 도구를 능동적으로 호출할 수 있게 됩니다. 이 작업에서는 에이전트가 재고 검사를 수행할 수 있게 하는 몇 가지 도구를 추가할 것입니다.

1. 함수 코드용으로 제공된 코드 파일을 편집하려면 다음 명령을 입력합니다.

    ```
   code server.py
    ```

    이 코드 파일에서는 에이전트가 소매점의 백 엔드 서비스를 시뮬레이트하는 데 사용할 수 있는 도구를 정의합니다. 파일의 맨 위에 있는 서버 설정 코드를 확인합니다. 해당 코드는 `FastMCP`을(를) 사용하여 ‘재고’로 명명된 MCP 서버 인스턴스를 빠르게 실행합니다. 이 서버는 사용자가 정의한 도구를 호스트하고 에이전트가 랩 도중 해당 도구에 액세스할 수 있게 합니다.

1. **Add an inventory check tool** 주석을 찾아 다음 코드를 추가합니다.

    ```python
   # Add an inventory check tool
   @mcp.tool()
   def get_inventory_levels() -> dict:
        """Returns current inventory for all products."""
        return {
            "Moisturizer": 6,
            "Shampoo": 8,
            "Body Spray": 28,
            "Hair Gel": 5, 
            "Lip Balm": 12,
            "Skin Serum": 9,
            "Cleanser": 30,
            "Conditioner": 3,
            "Setting Powder": 17,
            "Dry Shampoo": 45
        }
    ```

    이 사전은 샘플 재고를 나타냅니다. `@mcp.tool()` 주석을 사용하면 LLM에서 함수를 검색할 수 있습니다. 

1. **Add a weekly sales tool** 주석을 찾아 다음 코드를 추가합니다.

    ```python
   # Add a weekly sales tool
   @mcp.tool()
   def get_weekly_sales() -> dict:
        """Returns number of units sold last week."""
        return {
            "Moisturizer": 22,
            "Shampoo": 18,
            "Body Spray": 3,
            "Hair Gel": 2,
            "Lip Balm": 14,
            "Skin Serum": 19,
            "Cleanser": 4,
            "Conditioner": 1,
            "Setting Powder": 13,
            "Dry Shampoo": 17
        }
    ```

1. 파일을 저장합니다(*CTRL+S*).

### MCP 클라이언트 구현

MCP 클라이언트는 MCP 서버가 도구를 검색하고 호출할 수 있도록 연결하는 구성 요소입니다. 이는 서버에 호스팅된 함수와 에이전트 간의 브리지로 간주할 수 있으며, 사용자 프롬프트에 응답하여 능동적으로 도구를 사용할 수 있게 합니다.

1. 다음 명령을 입력하여 클라이언트 코드 편집을 시작합니다.

    ```
   code client.py
    ```

    > **팁**: 코드 파일에 코드를 추가할 때 올바른 들여쓰기를 유지해야 합니다.

1. **Add references** 주석을 찾고 다음 코드를 추가하여 클래스를 가져옵니다.

    ```python
   # Add references
   from mcp import ClientSession, StdioServerParameters
   from mcp.client.stdio import stdio_client
   from azure.ai.agents import AgentsClient
   from azure.ai.agents.models import FunctionTool, MessageRole, ListSortOrder
   from azure.identity import DefaultAzureCredential
    ```

1. **Start the MCP server** 주석을 찾아 다음 코드를 추가합니다.

    ```python
   # Start the MCP server
   stdio_transport = await exit_stack.enter_async_context(stdio_client(server_params))
   stdio, write = stdio_transport
    ```

    표준 프로덕션 설정에서 서버는 클라이언트와 별도로 실행됩니다. 하지만 이 랩에서 클라이언트는 표준 입출력 전송을 사용하여 서버를 시작해야 합니다. 이렇게 하면 두 구성 요소 사이에 경량 통신 채널이 만들어져 로컬 개발 설정이 간소화됩니다.

1. **Create an MCP client session** 주석을 찾아 다음 코드를 추가합니다.

    ```python
   # Create an MCP client session
   session = await exit_stack.enter_async_context(ClientSession(stdio, write))
   await session.initialize()
    ```

    이렇게 하면 이전 단계의 입력 및 출력 스트림을 사용하여 새로운 클라이언트 세션이 만들어집니다. `session.initialize`을(를) 호출하면 세션이 MCP 서버에 등록된 도구를 검색하고 호출할 수 있도록 준비합니다.

1. **List available tools** 주석 아래에 다음 코드를 추가하여 클라이언트가 서버에 연결되어 있는지 확인합니다.

    ```python
   # List available tools
   response = await session.list_tools()
   tools = response.tools
   print("\nConnected to server with tools:", [tool.name for tool in tools]) 
    ```

    이제 클라이언트 세션을 Azure AI 에이전트와 함께 사용할 준비를 마쳤습니다.

### 에이전트에 MCP 도구 연결

이 작업에서는 AI 에이전트를 준비하고, 사용자 프롬프트를 수락하고, 함수 도구를 호출합니다.

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

1. **List tools available on the server** 주석 아래에 다음 코드를 추가합니다.

    ```python
   # List tools available on the server
   response = await session.list_tools()
   tools = response.tools
    ```

1. **Build a function for each tool** 주석 아래에 다음 코드를 추가합니다.

    ```python
   # Build a function for each tool
   def make_tool_func(tool_name):
        async def tool_func(**kwargs):
            result = await session.call_tool(tool_name, kwargs)
            return result
        
        tool_func.__name__ = tool_name
        return tool_func

   functions_dict = {tool.name: make_tool_func(tool.name) for tool in tools}
   mcp_function_tool = FunctionTool(functions=list(functions_dict.values()))
    ```

    이 코드는 MCP 서버에서 사용할 수 있는 도구를 AI 에이전트에서 호출할 수 있도록 동적으로 래핑합니다. 각 도구는 비동기 함수로 전환된 다음, 에이전트에서 사용할 수 있도록 `FunctionTool` 번들로 묶습니다.

1. **Create the agent** 주석을 찾아 다음 코드를 추가합니다.

    ```python
   # Create the agent
   agent = agents_client.create_agent(
        model=model_deployment,
        name="inventory-agent",
        instructions="""
        You are an inventory assistant. Here are some general guidelines:
        - Recommend restock if item inventory < 10  and weekly sales > 15
        - Recommend clearance if item inventory > 20 and weekly sales < 5
        """,
        tools=mcp_function_tool.definitions
   )
    ```

1. **Enable auto function calling** 주석을 찾아 다음 코드를 추가합니다.

    ```python
   # Enable auto function calling
   agents_client.enable_auto_function_calls(tools=mcp_function_tool)
    ```

1. **Create a thread for the chat session** 주석 아래에 다음 코드를 추가합니다.

    ```python
   # Create a thread for the chat session
   thread = agents_client.threads.create()
    ```

1. **Invoke the prompt** 주석을 찾아 다음 코드를 추가합니다.

    ```python
   # Invoke the prompt
   message = agents_client.messages.create(
        thread_id=thread.id,
        role=MessageRole.USER,
        content=user_input,
   )
   run = agents_client.runs.create(thread_id=thread.id, agent_id=agent.id)
    ```

1. **Retrieve the matching function tool** 주석을 찾아 다음 코드를 추가합니다.

    ```python
   # Retrieve the matching function tool
   function_name = tool_call.function.name
   args_json = tool_call.function.arguments
   kwargs = json.loads(args_json)
   required_function = functions_dict.get(function_name)

   # Invoke the function
   output = await required_function(**kwargs)
    ```

    이 코드는 에이전트 스레드의 도구 호출의 정보를 사용합니다. 함수 이름과 인수가 검색되고, 일치하는 함수를 호출하는 데 사용됩니다.

1. **Append the output text** 주석 아래에 다음 코드를 추가합니다.

    ```python
   # Append the output text
   tool_outputs.append({
        "tool_call_id": tool_call.id,
        "output": output.content[0].text,
   })
    ```

1. **Submit the tool call output** 주석 아래에 다음 코드를 추가합니다.

    ```python
   # Submit the tool call output
   agents_client.runs.submit_tool_outputs(thread_id=thread.id, run_id=run.id, tool_outputs=tool_outputs)
    ```

    이 코드는 필요한 작업이 완료되었음을 에이전트 스레드에 알리고 도구 호출 출력을 업데이트합니다.

1. **Display the response** 주석을 찾아 다음 코드를 추가합니다.

    ```python
   # Display the response
   messages = agents_client.messages.list(thread_id=thread.id, order=ListSortOrder.ASCENDING)
   for message in messages:
        if message.text_messages:
            last_msg = message.text_messages[-1]
            print(f"{message.role}:\n{last_msg.text.value}\n")
    ```

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

1. 메시지가 표시되면 다음과 같은 쿼리를 입력합니다.

    ```
   What are the current inventory levels?
    ```

    > **팁**: 속도 제한을 초과하여 앱이 실패하는 경우. 몇 초 정도 기다렸다가 다시 시도하세요. 구독에서 사용할 수 있는 할당량이 부족한 경우 모델이 응답하지 않을 수 있습니다.

    다음과 비슷한 결과가 나타나야 합니다.

    ```
    MessageRole.AGENT:
    Here are the current inventory levels:

    - Moisturizer: 6
    - Shampoo: 8
    - Body Spray: 28
    - Hair Gel: 5
    - Lip Balm: 12
    - Skin Serum: 9
    - Cleanser: 30
    - Conditioner: 3
    - Setting Powder: 17
    - Dry Shampoo: 45
    ```

1. 원하는 경우 대화를 계속할 수 있습니다. 스레드는 *상태 저장*이므로 대화 내용을 유지합니다. 즉, 에이전트에 각 응답에 대한 전체 컨텍스트가 있습니다. 

    다음과 같은 프롬프트를 입력해 보세요.

    ```
   Are there any products that should be restocked?
    ```

    ```
   Which products would you recommend for clearance?
    ```

    ```
   What are the best sellers this week?
    ```

    완료되면 `quit`을(를) 입력합니다.

## 정리

연습을 마쳤으므로 불필요한 리소스 사용을 방지하기 위해 만든 클라우드 리소스를 삭제해야 합니다.

1. [Azure Portal](https://portal.azure.com)을 `https://portal.azure.com`에서 열고 이 연습에서 사용한 허브 리소스를 배포한 리소스 그룹의 내용을 확인합니다.
1. 도구 모음에서 **리소스 그룹 삭제**를 선택합니다.
1. 리소스 그룹 이름을 입력하고 삭제할 것인지 확인합니다.
