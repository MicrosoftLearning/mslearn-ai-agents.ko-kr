---
lab:
  title: Microsoft 에이전트 프레임워크로 다중 에이전트 솔루션 개발하기
  description: Microsoft 에이전트 프레임워크 SDK를 사용하여 여러 에이전트가 공동 작업하도록 구성하는 방법 알아보기
---

# 다중 에이전트 솔루션 개발

이 연습에서는 Microsoft 에이전트 프레임워크 SDK에서 순차 오케스트레이션 패턴을 사용하는 방법을 연습합니다. 고객 피드백을 처리하고 다음 단계를 제안하기 위해 협력하는 세 개의 에이전트로 구성된 간단한 파이프라인을 만듭니다. 만들어 볼 에이전트는 다음과 같습니다.

- 요약 작성기 에이전트는 가공되지 않은 피드백을 짧고 중립적인 문장으로 압축합니다.
- 분류자 에이전트는 피드백을 긍정, 부정 또는 기능 요청으로 분류합니다.
- 마지막으로 권장 작업 에이전트는 적절한 후속작업 단계를 권장합니다.

Microsoft 에이전트 프레임워크 SDK를 사용하여 문제를 분석하고, 올바른 에이전트를 통해 경로 지정하고, 실행 가능한 결과를 생성하는 방법을 알아봅니다. 그럼 시작하겠습니다.

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

1. 왼쪽 탐색 창에서 **모델 및 엔드포인트**를 선택하고 **GPT-4o** 배포를 선택합니다.

1. **설정** 창에서 모델 배포의 이름을 기록합니다(**GPT-4o**이어야 함). **모델 및 엔드포인트** 페이지에서 배포를 확인하면 이를 확인할 수 있습니다(왼쪽 탐색 창에서 해당 페이지를 열면 됩니다).
1. 왼쪽 탐색 창에서 **개요**를 선택하면 다음과 같은 프로젝트의 메인 페이지가 표시됩니다.

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
   pip install azure-identity agent-framework
    ```

1. 제공된 구성 파일을 편집하려면 다음 명령을 입력합니다.

    ```
   code .env
    ```

    코드 편집기에서 파일이 열립니다.

1. 코드 파일에서 **your_openai_endpoint** 자리 표시자를 프로젝트의 엔드포인트로 바꿉니다(Azure AI Foundry 포털의 프로젝트 **개요** 페이지에서 복사). **your_model_deployment** 자리 표시자를 GPT-4o 모델 배포에 할당한 이름으로 바꿉니다.

1. 자리 표시자를 바꾼 후 **Ctrl+S** 명령을 사용하여 변경 내용을 저장한 다음 **Ctrl+Q** 명령을 사용하여 Cloud Shell 명령줄을 열어 두고 코드 편집기를 닫습니다.

### AI 에이전트 만들기

이제 다중 에이전트 솔루션을 위한 에이전트를 만들 준비가 되었습니다. 그럼 시작하겠습니다.

1. 다음 명령을 입력하여 **agents.py** 파일을 편집합니다.

    ```
   code agents.py
    ```

1. 파일 상단의 **참조 추가하기** 주석 아래에 다음 코드를 추가하여 에이전트를 구현하는 데 필요한 라이브러리의 네임스페이스를 참조합니다.

    ```python
   # Add references
   import asyncio
   from typing import cast
   from agent_framework import ChatMessage, Role, SequentialBuilder, WorkflowOutputEvent
   from agent_framework.azure import AzureAIAgentClient
   from azure.identity import AzureCliCredential
    ```

1. **main** 함수에서 잠시 에이전트 지침을 검토합니다. 이러한 지침은 오케스트레이션에서 각 에이전트의 동작을 정의합니다.

1. **채팅 클라이언트 만들기** 주석 아래에 다음 코드를 추가합니다.

    ```python
   # Create the chat client
   credential = AzureCliCredential()
   async with (
       AzureAIAgentClient(async_credential=credential) as chat_client,
   ):
    ```

    **AzureCliCredential** 개체를 사용하면 코드에서 Azure 계정에 인증할 수 있습니다. **AzureAIAgentClient** 개체는 .env 구성의 Azure AI Foundry 프로젝트 설정을 자동으로 포함합니다.

1. **에이전트 만들기** 주석 아래에 다음 코드를 추가합니다.

    (들여쓰기 수준을 유지해야 합니다.)

    ```python
   # Create agents
   summarizer = chat_client.create_agent(
       instructions=summarizer_instructions,
       name="summarizer",
   )

   classifier = chat_client.create_agent(
       instructions=classifier_instructions,
       name="classifier",
   )

   action = chat_client.create_agent(
       instructions=action_instructions,
       name="action",
   )
    ```

## 순차 오케스트레이션 만들기

1. **main** 함수에서 **현재 피드백 초기화** 주석을 찾아 다음 코드를 추가합니다.
    
    (들여쓰기 수준을 유지해야 합니다.)

    ```python
   # Initialize the current feedback
   feedback="""
   I use the dashboard every day to monitor metrics, and it works well overall. 
   But when I'm working late at night, the bright screen is really harsh on my eyes. 
   If you added a dark mode option, it would make the experience much more comfortable.
   """
    ```

1. **순차 오케스트레이션 빌드하기** 주석 아래에 다음 코드를 추가하여 정의한 에이전트로 순차 오케스트레이션을 정의합니다.

    ```python
   # Build sequential orchestration
   workflow = SequentialBuilder().participants([summarizer, classifier, action]).build()
    ```

    에이전트는 오케스트레이션에 추가된 순서대로 피드백을 처리합니다.

1. **출력 실행 및 수집** 주석 아래에 다음 코드를 추가합니다.

    ```python
   # Run and collect outputs
   outputs: list[list[ChatMessage]] = []
   async for event in workflow.run_stream(f"Customer feedback: {feedback}"):
       if isinstance(event, WorkflowOutputEvent):
           outputs.append(cast(list[ChatMessage], event.data))
    ```

    이 코드는 오케스트레이션을 실행하고 참여하는 각 에이전트가 출력한 내용을 수집합니다.

1. **출력 표시** 주석 아래에 다음 코드를 추가합니다.

    ```python
   # Display outputs
   if outputs:
       for i, msg in enumerate(outputs[-1], start=1):
           name = msg.author_name or ("assistant" if msg.role == Role.ASSISTANT else "user")
           print(f"{'-' * 60}\n{i:02d} [{name}]\n{msg.text}")
    ```

    이 코드는 오케스트레이션에서 수집한 워크플로 출력의 메시지를 형식 지정하고 표시합니다.

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
   python agents.py
    ```

    다음과 비슷한 결과가 나타나야 합니다.

    ```output
    ------------------------------------------------------------
    01 [user]
    Customer feedback:
        I use the dashboard every day to monitor metrics, and it works well overall.
        But when I'm working late at night, the bright screen is really harsh on my eyes.
        If you added a dark mode option, it would make the experience much more comfortable.

    ------------------------------------------------------------
    02 [summarizer]
    User requests a dark mode for better nighttime usability.
    ------------------------------------------------------------
    03 [classifier]
    Feature request
    ------------------------------------------------------------
    04 [action]
    Log as enhancement request for product backlog.
    ```

1. 필요에 따라 다음과 같은 다른 피드백 입력을 사용하여 코드를 실행해 볼 수 있습니다.

    ```output
    I use the dashboard every day to monitor metrics, and it works well overall. But when I'm working late at night, the bright screen is really harsh on my eyes. If you added a dark mode option, it would make the experience much more comfortable.
    ```
    ```output
    I reached out to your customer support yesterday because I couldn't access my account. The representative responded almost immediately, was polite and professional, and fixed the issue within minutes. Honestly, it was one of the best support experiences I've ever had.
    ```

## 요약

이번 연습에서는 Microsoft 에이전트 프레임워크 SDK로 순차 오케스트레이션을 연습하여 여러 에이전트를 간소화된 단일 워크플로로 결합해 보았습니다. 잘했습니다!

## 정리

Azure AI 에이전트 서비스 탐색을 마친 경우 이 연습에서 만든 리소스를 삭제하여 불필요한 Azure 비용이 발생하지 않도록 해야 합니다.

1. Azure Portal이 포함된 브라우저 탭으로 돌아가서(또는 새 브라우저 탭의 `https://portal.azure.com`에서 [Azure Portal](https://portal.azure.com)을 다시 열고) 이 연습에 사용된 리소스를 배포한 리소스 그룹의 콘텐츠를 확인합니다.

1. 도구 모음에서 **리소스 그룹 삭제**를 선택합니다.

1. 리소스 그룹 이름을 입력하고 삭제할 것인지 확인합니다.
