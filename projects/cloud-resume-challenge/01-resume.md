# Resume

이번 장에선 AWS 상에서 배포 대상이 될 이력서 작성 방법을 설명드리겠습니다.

사용 기술: HTML, CSS

## HTML, CSS

이력서에 들어갈 내용을 작성하는 도전 과제입니다. Word 혹은 PDF 형태가 아닌 HTML, CSS로 작성하는 제한 조건이 걸려있습니다. 이는 이후 Javascript와 AWS Lambda를 활용하여 '방문자 수 카운트' 기능을 구현하기위함입니다. CRC를 시작한 목적이 AWS와 DevOps 학습이 주된 목적이고 이번에 작성할 이력서는 실제로 인터넷 상에 공개할 예정이었기 때문에 인터넷 상에 공개된 이상적인 템플릿 찾아 수정할 계획을 세웠습니다.

이력서 템플릿을 구글링 해 보니 가장 이상적인 템플릿으로는 [Harvard Template](https://careerservices.fas.harvard.edu/channels/create-a-resume-cv-or-cover-letter/) 과 [WSO](https://www.wallstreetoasis.com/resources/templates/word-templates/investment-banking-resume-template)템플릿이 언급됐습니다. 그러나, 두 템플릿 모두 Word 파일로 제공되어 바로 웹에 사용하는 것은 무리가 있었습니다. 그래서 이러한 템플릿과 최대한 유사한 템플릿을 Github에서 검색하였습니다. 종이로 인쇄하여 제출할 것 까지 생각하여 동적인 이력서는 처음부터 배제하였고, 심플한 디자인으로 구성된 템플릿을 선별을 거듭한 끝에 [mikepqr/resume.md](https://github.com/mikepqr/resume.md)를 선택하였습니다(레포지토리의 설명에 따르면, 해당 템플릿은 [The Tech Resume Inside-Out](https://www.thetechinterview.com/)라는 책에서 선정한 이상적인 이력서 템플릿 중 하나로 언급하고 있음을 설명합니다).

![mikepqr/resume.md 템플릿](https://img.yibyeongyong.com/crc/01-resume/00-generic-resume.png)

HTML과 CSS를 깊게 공부한 적은 없지만, 한글 폰트 적용, 폰트 사이즈 수정 등의 필요한 기능을 인터넷에서 찾아 기존 이력서에 적용하였습니다. 이력서의 내용은 한 장 이내로 작성하고, 간결한 문구로 작성하는 것을 염두하며 작성하였습니다.
