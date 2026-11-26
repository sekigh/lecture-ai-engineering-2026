# 第6回 解答

## 演習

- STEP 1 のツールの名前：[glossary_search_glossary]
- STEP 2 の [tool call]： glossary_search_glossary({'query': '3レベル'})
- STEP 3 の [tool call]： fetch_fetch({'url': 'https://modelcontextprotocol.io/'})
- STEP 4 の [tool call]：aws_aws___search_documentation({'search_phrase': 'Amazon S3 versioning overview'})
- 気づき・疑問：自分のtoolをMCPサーバ化しlocalで実行する際は、mcp_servers.jsonにcomandをuvで記述するが、外部のMCPサーバを呼び込んでlocalで実行する際は、commandがuvxとすることで違いがあることが分かった。uvは、localの実行環境を明示的に構築し自家製MCPサーバを実行するが、uvxは、localの環境を表向きは書き換えずに（ただしキャッシュに書き込みは行い、同じMCPサーバが連続して呼ばれた場合にレスポンスは速い）、仮設的に外部MCPサーバを実行する環境を作り、実行することが分かった。外部組織が管理運営するMCPサーバーは、そのMCPサーバ名をmcp_servers.jsonに登録することでよく、commanの登録は無しである点が分かった。

## 宿題

- 取り組んだレベル：基礎 および 実装 
- やったこと：
  ・基礎
  　app.tools.pyに既に組み込まれているtool:get_lecture_summary()を自前のMCPサーバとして利用する。app.mcp_server.pyに、server.tool()(tools.get_lecture_summary)を追加した。
  ・実装
  　演習３で自作したtool:make_quiz()を自前のMCPサーバとして利用する。app.mcp_server.pyに、server.tool()(toos.make_quiz)を追加した。講義回が存在しない回が指定された場合は、make_quiz()が返り値として、「"第{lecture}回に対応する用語は見つかりませんでした。」と返すので、LLMがそれを受けて、何らかの回答を作ることを期待する。
- 結果：
　・基礎
「 $ uv run python -m app.mcp_agent --trace "第3回の講義内容を3行以内で要約して"」を実行し、正しく、第3回の講義要約を3行程度で出力した。[tool call]   glossary_get_lecture_summary({'lecture': 3})
また、「3行以内」を省いて、「 $ uv run python -m app.mcp_agent --trace "第3回の講義内容で要約して"」を実行すると、20行程度の要約を出力した。[tool call]   glossary_get_lecture_summary({'lecture': 3})

　・実装
　「 $ uv run python -m app.mcp_agent --trace "第2回の内容から確認問題を1問出して"」で、「**【確認問題】**　Q. 「ファインチューニング」の意味は？」と、正しい質問を出力した。[tool call]   glossary_make_quiz({'lecture': 2})
また、講義回がない回を指定した場合、「 $ uv run python -m app.mcp_agent --trace "第99回の内容から確認問題を1問出して"」では、「第99回に関する情報や用語は講座の用語集に登録されていないため、確認問題を作成することができません。現在の講座で用意されている回の範囲内で確認問題を出題することは可能ですので、ご希望の回（例えば第1回〜第N回など）を指定していただければ作成いたします。」と丁寧に再質問を促した。上手くいった。[tool call]   glossary_make_quiz({'lecture': 99})
- 気づき・疑問：localに作成した２つのtoolをMCPサーバ化した。localのツールの場合、mcp_servers.jsonには、local tool用に、既にglossary名でMCPサーバが登録されているので、追加の２つのツールのために、特に記述を追加する必要はないことを理解した。