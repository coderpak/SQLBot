# locales 目录教程

- `zh-CN.json`：简体中文业务文案。
- `zh-TW.json`：繁体中文业务文案。
- `en.json`：英文业务文案。
- `ko-KR.json`：韩文业务文案。

它们服务运行期业务消息；Swagger 文案位于 `apps/swagger/locales`。新增 key 时应为所有语言提供值或确认回退策略。

`common/utils/locale.py` 根据 Accept-Language/用户设置选择语言，`Trans` 依赖把翻译器注入 API。业务层应传稳定 key 和参数，不要在代码里拼接中英文句子。前端只负责显示后端消息，不应再次把已翻译文本当 key 翻译。

