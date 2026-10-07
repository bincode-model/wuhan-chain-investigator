# 官方浏览器验收适配限制

2026-10-07，本次继续使用用户选定的完整交付 Harness，未移除发布门禁。

当前 canonical 1.0.0 的报告 runner 只支持 unittest；UI contract 必须关联非 mock E2E，fresh attestation 又在禁止网络的沙箱重跑 Python。官方 cua_repl 的真实交互可归档为证据，但没有导入受信浏览器执行收据的接口。读取历史浏览器记录的 Python 检查不能冒充重新执行 E2E。

因此，即便常规 lint、typecheck、build、后端回归与官方浏览器检查成功，也不等于完整 Harness PASS。release_allowed 必须保持 false，直到 canonical 工具补齐真实受信浏览器 runner，或用户明确另行决定交付方式。

保留初次前端 worker 的 BLOCKED 生命周期记录，不修饰或删除。该记录包含工作区尚未提交、任务状态与未执行验收不符两项问题；源码产物需要独立检查。

此文档描述交付工具限制，不是产品对外界面内容。
