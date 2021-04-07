# Mednafen for PlayStation

## Pattern
* In `state` attach to Mednafen
* In `init`:
  * Signature scan for an intruction containing a pointer to MainRAM
  * Read the pointer from the instruction and use it as base

## Example
```c#
state("mednafen") {
}

startup {
  vars.BaseAddress = IntPtr.Zero;
}

init {
  vars.BaseAddress = IntPtr.Zero;
  
  var module = modules.First();
  var signature = "48 c7 44 24 ?? ?? ?? ?? ?? 48 c7 44 24 ?? ?? ?? ?? ?? c7 44 24 ?? 00 00 20 00";
    
  var signatureTarget = new SigScanTarget(5, signature);
  var signatureScanner = new SignatureScanner(game, module.BaseAddress, (int)module.ModuleMemorySize);
  IntPtr codeOffset = signatureScanner.Scan(signatureTarget);
  
  if (codeOffset != IntPtr.Zero) {
    vars.BaseAddress = (IntPtr)memory.ReadValue<int>(codeOffset);
  }
  
  if (vars.BaseAddress != IntPtr.Zero) {
    
    // Set up watchers
    vars.Watchers = new MemoryWatcherList() {
      new MemoryWatcher<byte>(IntPtr.Add(vars.BaseAddress, 0x123456)) { Name = "ByteValue" },
      new StringWatcher(IntPtr.Add(vars.BaseAddress, 0x789ABC), 123) { Name = "StringValue" }
    };
  
  }
}

// Example blocks
update {
  vars.Watchers.UpdateAll();
}

split {
  return (vars.Watchers["StringValue"].Changed);
}
```
