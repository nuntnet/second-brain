---
name: reference_browser_pane_file_upload_via_datatransfer
description: Browser pane can't drive the OS file picker — attach a file in the AI chat composer by setting input[type=file].files via DataTransfer and dispatching change
metadata:
  type: reference
---

Seen 2026-10-29 (AI-109, local console :5191). The composer's attach button opens an OS picker the pane cannot use. This runs the real upload XHR:

```js
const inp=document.querySelector('input[type=file]');
const f=new File([new Uint8Array([0x89,0x50,0x4e,0x47,13,10,26,10])],'photo.png',{type:'image/png'});
const dt=new DataTransfer(); dt.items.add(f); inp.files=dt.files;
inp.dispatchEvent(new Event('change',{bubbles:true}));
```

Then read `[data-testid=composer-attachment-error]`. A fake PNG renders a broken thumbnail, which is not a defect. Conversation route is `/w/<ws>?session=<id>`; `/w/<ws>/inbox/<id>` is a 404.
Related: [[reference_local_ai_chat_console_open_on_localhost_with_seed_identity]].
