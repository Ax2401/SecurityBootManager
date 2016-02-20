# Grub 시작 문자열 구성 변경 및 함수 호출 스택
--------------------------------------------------------------------------

- Grub의 처음 시작이 되는 메인 GUI 화면의 텍스트 문구 수정 및 새로운 GUI 기반의 페이지 추가
https://github.com/IWillFindYou/SecurityBootManager/tree/develop/grub/grub-core/normal/menu_text.c#L153

- 해당 파일의 print_message() 함수를 기반으로 함수 호출 스택을 분석할 것!!
아래 이슈로 지속적인 도움이 될만한 분석자료들을 추가 요망

- 해당 normal 모드의 Screen 초기화 루틴 부분
https://github.com/IWillFindYou/SecurityBootManager/tree/develop/grub/grub-core/normal/main.c#L200

- 운영체제 선택부분에 대한 화면 그리는 페이지 이 함수안에 선 그리는 함수등이 사용되어져 있음
https://github.com/IWillFindYou/SecurityBootManager/tree/develop/grub/grub-core/normal/menu_text.c#L327

- 운영체제 선택 화면에서 키보드 입력에 따른 모드 전환 부분에 대한 코드가 존재
https://github.com/IWillFindYou/SecurityBootManager/tree/develop/grub/grub-core/normal/menu.c#L784

###*분석 중 참고해야할 것들*
[./grub/grub-core/normal/menu.c#L577](https://github.com/IWillFindYou/SecurityBootManager/tree/develop/grub/grub-core/normal/menu.c#L577)
기준으로 Grub System 시작됨

함수의 위치는 최상단의 검색 이용할 것

[./grub/grub-core/boot/i386/pc/boot.S#L455](https://github.com/IWillFindYou/SecurityBootManager/tree/develop/grub/grub-core/boot/i386/pc/boot.S#L455)에서
[./grub/include/grub/i386/pc/boot.h#L63](https://github.com/IWillFindYou/SecurityBootManager/tree/develop/grub/include/grub/i386/pc/boot.h#L63) 의 상수를 사용하여 점프하게 되는 커널 주소를 셋팅

[./grub/grub-core/boot/i386/pc/startup_raw.S#L285](https://github.com/IWillFindYou/SecurityBootManager/tree/develop/grub/grub-core/boot/i386/pc/startup_raw.S#L285)에서 multiboot_entry가 시작됨
**이미지 파일 생성시 참조되는 기초 파일들은 [./grub/grub-core/Makefile.core.def](https://github.com/IWillFindYou/SecurityBootManager/tree/develop/grub/grub-core/Makefile.core.def) 참조**

이후 커널의 메인은 아래에서 호출됨
[.grub/grub-core/kern/i386/pc/startup.S#L124](https://github.com/IWillFindYou/SecurityBootManager/tree/develop/grub/grub-core/kern/i386/pc/startup.S#L124)

grub_main() 함수에서 grub_dl_load() 함수를 이용하여 "normal" 에대한 함수를 등록후 이후 grub_command_execute() inline 함수를 이용하여 실행하는 것을 확인
[./grub/grub-core/kern/dl.c#L737](https://github.com/IWillFindYou/SecurityBootManager/tree/develop/grub/grub-core/kern/dl.c#L737)
[./grub/include/grub/dl.h#L222](https://github.com/IWillFindYou/SecurityBootManager/tree/develop/grub/include/grub/dl.h#L222)

 grub_dl_load() 함수의 경우 elf 파일 format 을 바이너리 분석을 통해
normal.mod 라는 모드 파일을 찾아 함수 포인터를 로드 이후 GRUB_MOD_INIT() 을 통해 정의된 함수의 초기화 루틴 호출하는 방식을 통해 호출

모듈의 초기화 루틴 즉 grub_main() 함수로 부터 normal 루틴의 run_menu() 함수가 호출되기 까지의 과정은 아래와 같음

별도의 링크를 적어논 것은 검색으로 찾을 수 없는 함수들 이므로 링크를 참조하기 바람.

###*함수 호출 스택*
[grub_main](https://github.com/IWillFindYou/SecurityBootManager/tree/develop/grub/grub-core/kern/main.c#L265)() -> [grub_load_normal_mode](https://github.com/IWillFindYou/SecurityBootManager/tree/develop/grub/grub-core/kern/main.c#L227)() -> [grub_dl_load("normal")](https://github.com/IWillFindYou/SecurityBootManager/tree/develop/grub/grub-core/kern/main.c#L230) -> [grub_dl_load_file](https://github.com/IWillFindYou/SecurityBootManager/tree/develop/grub/grub-core/kern/dl.c#L684)() ->
[grub_dl_load_core](https://github.com/IWillFindYou/SecurityBootManager/tree/develop/grub/grub-core/kern/dl.c#L664)() -> [grub_dl_load_core_noinit](https://github.com/IWillFindYou/SecurityBootManager/tree/develop/grub/grub-core/kern/dl.c#L598)() -> [grub_dl_resolve_symbols](https://github.com/IWillFindYou/SecurityBootManager/tree/develop/grub/grub-core/kern/dl.c#L330)() ->
해당 모듈의 grub_mod_init() 함수 포인터를 셋팅 ->
[grub_dl_init](https://github.com/IWillFindYou/SecurityBootManager/tree/develop/grub/include/grub/dl.h#L222)("normal") -> normal 모듈의 [GRUB_MOD_INIT(normal)](https://github.com/IWillFindYou/SecurityBootManager/tree/develop/grub/grub-core/normal/main.c#L524) 호출 ->
[grub_register_command](https://github.com/IWillFindYou/SecurityBootManager/tree/develop/grub/grub-core/normal/main.c#L556)("normal")을 실행하여 [grub_cmd_normal](https://github.com/IWillFindYou/SecurityBootManager/tree/develop/grub/grub-core/normal/main.c#L314)() 함수를 등록 ->
[grub_load_normal_mode](https://github.com/IWillFindYou/SecurityBootManager/tree/develop/grub/grub-core/kern/main.c#L227)() 함수로 되돌아옴 -> [grub_command_execute](https://github.com/IWillFindYou/SecurityBootManager/tree/develop/grub/grub-core/kern/main.c#L237)("normal") ->
등록된 normal 명령어의 [grub_cmd_normal](https://github.com/IWillFindYou/SecurityBootManager/tree/develop/grub/grub-core/normal/main.c#L344)() 함수 실행 -> [grub_enter_normal_mode](https://github.com/IWillFindYou/SecurityBootManager/tree/develop/grub/grub-core/normal/main.c#L299)() ->
[grub_normal_execute](https://github.com/IWillFindYou/SecurityBootManager/tree/develop/grub/grub-core/normal/main.c#L260)() 실행 후 만약 [grub_cmdline_run](https://github.com/IWillFindYou/SecurityBootManager/tree/develop/grub/grub-core/normal/main.c#L457)() 실행할 경우 커멘드 라인 실행 ->
[grub_show_menu](https://github.com/IWillFindYou/SecurityBootManager/tree/develop/grub/grub-core/normal/menu.c#L887)() -> [show_menu](https://github.com/IWillFindYou/SecurityBootManager/tree/develop/grub/grub-core/normal/menu.c#L856)() ->
[run_menu](https://github.com/IWillFindYou/SecurityBootManager/tree/develop/grub/grub-core/normal/menu.c#L577)() -> [grub_menu_entry_run](https://github.com/IWillFindYou/SecurityBootManager/tree/develop/grub/grub-core/normal/menu_entry.c#L1228)() -> [grub_menu_init_page](https://github.com/IWillFindYou/SecurityBootManager/tree/develop/grub/grub-core/normal/menu_text.c#L327)() -> **[print_message](https://github.com/IWillFindYou/SecurityBootManager/tree/develop/grub/grub-core/normal/menu_text.c#L153)()**
