---
lab:
  title: Microsoft 에이전트 프레임워크 SDK로 Azure AI 에이전트 개발하기
  description: Microsoft 에이전트 프레임워크 SDK를 사용하여 Azure AI 채팅 에이전트를 만들고 사용하는 방법을 알아봅니다.
---

# Microsoft 에이전트 프레임워크 SDK로 Azure AI 채팅 에이전트 개발하기

이 연습에서는 Azure AI 에이전트 서비스 및 Microsoft 에이전트 프레임워크를 사용하여 비용 클레임을 처리하는 AI 에이전트를 만듭니다.

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
    - **지역**: ***AI Foundry 권장 사항 선택***\*

    > \* 일부 Azure AI 리소스는 지역 모델 할당량에 의해 제한됩니다. 연습 후반부에 할당량 한도를 초과하는 경우 다른 지역에서 다른 리소스를 만들어야 할 수도 있습니다.

1. **만들기**를 선택하고 선택한 GPT-4 모델 배포를 포함한 프로젝트가 생성될 때까지 기다립니다.
1. 프로젝트를 만들면 채팅 플레이그라운드가 자동으로 열립니다.
1. **설정** 창에서 모델 배포의 이름을 기록합니다(**GPT-4o**이어야 함). **모델 및 엔드포인트** 페이지에서 배포를 확인하면 이를 확인할 수 있습니다(왼쪽 탐색 창에서 해당 페이지를 열면 됩니다).
1. 왼쪽 탐색 창에서 **개요**를 선택하면 다음과 같은 프로젝트의 메인 페이지가 표시됩니다.

    ![Azure AI 파운드리 포털의 Azure AI 프로젝트 세부 정보 스크린샷.](./Media/ai-foundry-project.png)

## 에이전트 클라이언트 앱 만들기

이제 에이전트와 사용자 지정 함수를 정의하는 클라이언트 앱을 만들 준비가 되었습니다. GitHub 리포지토리에 일부 코드가 제공되었습니다.

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

    > **팁**: CloudShell에 명령을 입력할 때 출력이 화면 버퍼를 많이 차지하여 현재 줄의 커서가 가려질 수 있습니다. `cls` 명령을 입력해 화면을 지우면 각 작업에 더 집중할 수 있습니다.

1. 리포지토리가 복제된 경우 다음 명령을 입력하여 작업 디렉터리를 코드 파일이 포함된 폴더로 변경하고 모두 나열합니다.

    ```
   cd ai-agents/Labfiles/04-agent-framework/python
   ls -a -l
    ```

    제공된 파일에는 애플리케이션 코드, 구성 설정 파일, 비용 데이터가 포함된 파일이 포함됩니다.

### 애플리케이션 설정을 구성합니다.

1. Cloud Shell 명령줄 창에서 다음 명령을 입력하여 사용할 라이브러리를 설치합니다.

    ```
   python -m venv labenv
   ./labenv/bin/Activate.ps1
   pip install azure-identity agent-framework
    ```

1. 제공된 구성 파일을 편집하려면 다음 명령을 입력합니다.

    ```
   code .env
    ```

    코드 편집기에서 파일이 열립니다.

1. 코드 파일에서 **your_project_endpoint** 자리 표시자를 프로젝트의 엔드포인트(Azure AI 파운드리 포털의 프로젝트 **개요** 페이지에서 복사)로 바꾸고 **your_model_deployment** 자리 표시자를 GPT-4o 모델 배포에 할당한 이름으로 바꿉니다.
1. 자리 표시자를 바꾼 후 **Ctrl+S** 명령을 사용하여 변경 내용을 저장한 다음 **Ctrl+Q** 명령을 사용하여 Cloud Shell 명령줄을 열어 두고 코드 편집기를 닫습니다.

### 에이전트 앱에 대한 코드 작성

> **팁**: 코드를 추가할 때 올바른 들여쓰기를 유지해야 합니다. 기존 주석을 가이드로 사용하여 동일한 수준의 들여쓰기에서 새 코드를 입력합니다.

1. 제공된 에이전트 코드 파일을 편집하려면 다음 명령을 입력합니다.

    ```
   code agent-framework.py
    ```

1. 이 파일의 코드를 검토합니다. 여기에는 다음이 포함됩니다.
    - 일반적으로 사용되는 네임스페이스에 대한 참조를 추가하는 일부 **import** 문
    - 비용 데이터가 포함된 파일을 로드하고 사용자에게 지침을 요청한 다음 호출하는 *main* 함수입니다.
    - 에이전트를 만들고 사용하는 코드를 추가해야 하는 **process_expenses_data** 함수

1. 파일 상단에서 기존 **import** 문 뒤의 **Add references** 주석을 찾아 다음 코드를 추가하여 에이전트를 구현하는 데 필요한 라이브러리의 네임스페이스를 참조합니다.

    ```python
   # Add references
   from agent_framework import AgentThread, ChatAgent
   from agent_framework.azure import AzureAIAgentClient
   from azure.identity.aio import AzureCliCredential
   from pydantic import Field
   from typing import Annotated
    ```

1. 파일 하단에서 **이메일 기능을 위한 도구 함수 만들기** 주석을 찾아 다음 코드를 추가하여 에이전트가 이메일을 보내는 데 사용할 함수를 정의합니다(도구는 에이전트에 사용자 지정 기능을 추가하는 방법입니다).

    ```python
   # Create a tool function for the email functionality
   def send_email(
    to: Annotated[str, Field(description="Who to send the email to")],
    subject: Annotated[str, Field(description="The subject of the email.")],
    body: Annotated[str, Field(description="The text body of the email.")]):
        print("\nTo:", to)
        print("Subject:", subject)
        print(body, "\n")
    ```

    > **참고**: 이 함수는 이메일을 콘솔에 출력하여 이메일 전송을 *시뮬레이션*합니다. 실제 애플리케이션에서는 SMTP 서비스 또는 이와 유사한 서비스를 사용하여 실제로 이메일을 보냅니다.

1. **send_email** 코드 위쪽 부분을 백업하고, **process_expenses_data** 함수에서 **채팅 에이전트 만들기** 주석을 찾아 다음 코드를 추가하여 도구와 지침이 포함된 **ChatAgent** 개체를 만듭니다.

    (들여쓰기 수준을 유지해야 합니다.)

    ```python
   # Create a chat agent
   async with (
       AzureCliCredential() as credential,
       ChatAgent(
           chat_client=AzureAIAgentClient(async_credential=credential),
           name="expenses_agent",
           instructions="""You are an AI assistant for expense claim submission.
                           When a user submits expenses data and requests an expense claim, use the plug-in function to send an email to expenses@contoso.com with the subject 'Expense Claim`and a body that contains itemized expenses with a total.
                           Then confirm to the user that you've done so.""",
           tools=send_email,
       ) as agent,
   ):
    ```

    **AzureCliCredential** 개체를 사용하면 코드에서 Azure 계정에 인증할 수 있습니다. **AzureAIAgentClient** 개체는 .env 구성의 Azure AI Foundry 프로젝트 설정을 자동으로 포함합니다.

1. **Use the agent to process the expenses data** 주석을 찾아 다음 코드를 추가하여 에이전트가 실행할 스레드를 만든 다음 채팅 메시지로 호출합니다.

    (들여쓰기 수준을 유지 관리해야 합니다):

    ```python
   # Use the agent to process the expenses data
   try:
       # Add the input prompt to a list of messages to be submitted
       prompt_messages = [f"{prompt}: {expenses_data}"]
       # Invoke the agent for the specified thread with the messages
       response = await agent.run(prompt_messages)
       # Display the response
       print(f"\n# Agent:\n{response}")
   except Exception as e:
       # Something went wrong
       print (e)
    ```

1. 각 코드 블록의 기능을 이해하는 데 도움이 되는 주석을 사용하여 에이전트용의 완성된 코드를 검토한 다음 코드 변경 내용을 저장합니다(**CTRL+S**).
1. 코드의 오타를 수정해야 할 경우를 대비하여 코드 편집기를 열어 두되, 명령줄 콘솔을 더 많이 볼 수 있도록 창 크기를 조정합니다.

### Azure에 로그인하고 앱 실행

1. 코드 편집기 아래의 Cloud Shell 명령줄 창에서 다음 명령을 입력하여 Azure에 로그인합니다.

    ```
    az login
    ```

    **<font color="red">Cloud Shell 세션이 이미 인증되었더라도 Azure에 로그인해야 합니다.</font>**

    > **참고**: 대부분의 시나리오에서는 *az login*을 사용하는 것만으로도 충분합니다. 그러나 여러 테넌트에 구독이 있는 경우 *--tenant* 매개 변수를 사용하여 테넌트 지정해야 할 수 있습니다. 자세한 내용은 [Sign into Azure interactively using the Azure CLI](https://learn.microsoft.com/cli/azure/authenticate-azure-cli-interactively)를 참조하세요.
    
1. 메시지가 표시되면 지침에 따라 새 탭에서 로그인 페이지를 열고 제공된 인증 코드와 Azure 자격 증명을 입력합니다. 그런 다음 명령줄에서 로그인 프로세스를 완료하고 메시지가 표시되면 Azure AI 파운드리 허브가 포함된 구독을 선택합니다.
1. 로그인한 후 다음 명령을 입력하여 애플리케이션을 실행합니다.

    ```
   python agent-framework.py
    ```
    
    애플리케이션은 인증된 Azure 세션에 대한 자격 증명을 사용하여 실행하여 프로젝트에 연결하고 에이전트를 만들고 실행합니다.

1. 비용 데이터로 수행할 작업을 묻는 메시지가 표시되면 다음 프롬프트를 입력합니다.

    ```
   Submit an expense claim
    ```

1. 애플리케이션이 완료되면 출력을 검토합니다. 에이전트는 제공된 데이터를 기반으로 비용 청구 이메일을 작성했어야 합니다.

    > **팁**: 속도 제한을 초과하여 앱이 실패하는 경우. 몇 초 정도 기다렸다가 다시 시도하세요. 구독에서 사용할 수 있는 할당량이 부족한 경우 모델이 응답하지 않을 수 있습니다.

## 요약

이번 연습에서는 Microsoft 에이전트 프레임워크 SDK를 사용해 사용자 지정 도구로 에이전트를 만들어 보았습니다.

## 정리

Azure AI 에이전트 서비스 탐색을 마친 경우 이 연습에서 만든 리소스를 삭제하여 불필요한 Azure 비용이 발생하지 않도록 해야 합니다.

1. Azure Portal이 포함된 브라우저 탭으로 돌아가서(또는 새 브라우저 탭의 `https://portal.azure.com`에서 [Azure Portal](https://portal.azure.com)을 다시 열고) 이 연습에 사용된 리소스를 배포한 리소스 그룹의 콘텐츠를 확인합니다.
1. 도구 모음에서 **리소스 그룹 삭제**를 선택합니다.
1. 리소스 그룹 이름을 입력하고 삭제할 것인지 확인합니다.
