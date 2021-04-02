# BizHawk for PlayStation

BizHawk uses the Octoshock core, which is a port of Mednafen's PSX core.

## Pattern
* In `state` attach to BizHawk
* In `update`, if no memory has been found previously:
  * Signature scan in `octoshock.dll` for a relevant instruction in the [`shock_GetMemData`](https://github.com/TASVideos/BizHawk/blob/3ff0eb33dbd3073223467a02e9ec2928061968b8/psx/octoshock/psx/psx.cpp#L2632) function
  * Read the MainRAM pointer reference address from the instruction
  * Read the pointer to get the MainRAM base address

## Example
```c#
state("EmuHawk") {
}

startup {
  vars.BaseAddress = IntPtr.Zero;
}

init {
  vars.BaseAddress = IntPtr.Zero;
}

update {
  if (vars.BaseAddress == IntPtr.Zero) {
  
    var module = modules.Where(m => m.ModuleName == module).FirstOrDefault();
    if (module != null) {
      return false;
    }

    var signature = "49 03 c9 ff e1 48 8d 05 ?? ?? ?? ?? 48 89 02";

    var signatureTarget = new SigScanTarget(emu.SignatureOffset, emu.Signature);
    var signatureScanner = new SignatureScanner(game, module.BaseAddress, (int)module.ModuleMemorySize);
    IntPtr codeOffset = signatureScanner.Scan(signatureTarget);

    if (codeOffset != IntPtr.Zero) {
      int memoryReference = (int)((long)memory.ReadValue<int>(codeOffset) + (long)codeOffset + 4 -(long)module.BaseAddress);
      var deepPointer = new DeepPointer(emu.Module.ModuleName, memoryReference);

      IntPtr outOffset;
      deepPointer.DerefOffsets(game, out outOffset);
      var.BaseAddress = outOffset;
    }

    if (vars.BaseAddress == IntPtr.Zero) {
      return false;
    }

    // Set up watchers
    vars.Watchers = new MemoryWatcherList() {
      new MemoryWatcher<byte>(IntPtr.Add(vars.BaseAddress, 0x123456)) { Name = "ByteValue" },
      new StringWatcher(IntPtr.Add(vars.BaseAddress, 0x789ABC), 123) { Name = "StringValue" }
    };
    
  }
  
  // Game-specific logic below
  vars.Watchers.UpdateAll();
  
}

// Example blocks
split {
  return (vars.Watchers["StringValue"].Changed);
}
```
