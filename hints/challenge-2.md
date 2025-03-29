# Steps to defeat:

<details>
  <summary>Step 1</summary>
  
  Get all decryption function calls through AST parsing or regex `/_0x5630f6f4f99af80a\("([^"]+|)"\)/gm`.
</details>

<details>
  <summary>Step 2</summary>
  
  Replace all encrypted strings with their decrypted counterpart.
</details>

<details>
  <summary>Step 3</summary>
  
  Apply patch to generate flag.
</details>

<details>
  <summary>Step 4</summary>
  
  Evaluate the code.
</details>