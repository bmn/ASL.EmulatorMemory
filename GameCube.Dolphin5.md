# Dolphin 5 for GameCube

## Pattern
* In `state`, attach to Dolphin
* In `update`, validate any existing found memory. If not found:
  - Iterate through Dolphin's mapped regions of memory:
    + for a region of size 32MB (GC standard memory size)
    + beginning with the desired game ID
  - Set up watchers when memory is found

## Example
```c#
// Attach to Dolphin
state("Dolphin") {
}

// Set up default values
startup {
  vars.GameId = "ABCDE1";
  vars.BaseAddress = IntPtr.Zero;
  vars.Watchers = new MemoryWatcherList();
}


update {
  
  // If no existing found memory, attempt to find memory
  if (vars.BaseAddress == IntPtr.Zero) {
    // Iterate through all memory regions
    foreach (Memory page in game.MemoryPages(true))
    {
      // Check for mapped region size 32MB
      if ((page.RegionSize != (UIntPtr)0x2000000) || (page.Type != MemPageType.MEM_MAPPED)) {
        continue;
      }

      // Check game ID matches
      string gameId = memory.ReadString((IntPtr)page.BaseAddress, 6);
      if (!gameId.Equals(vars.GameId)) {
        continue;
      }
      vars.BaseAddress = page.BaseAddress;
      
      // Set up watchers
      vars.Watchers = new MemoryWatcherList() {
        // The game ID is used later to check the base address is still valid
        new StringWatcher(D.BaseAddress) { Name = "GameId" },
        
        // A simple address using D.BaseAddress as the base
        new MemoryWatcher<byte>((IntPtr)(D.BaseAddress + 0x123456)) { Name = "ByteValue" },
        
        // A deep pointer for addresses using multiple offsets
        new StringWatcher(
          new DeepPointer(D.BaseAddress, new Int32[] { 0x789ABC, 0xDEF012 })
        ) { Name = "StringValue" };
      };
      
      break;
    }
    
    return false;
  }
  
  // Update all values
  vars.Watchers.UpdateAll();
  
  // Check that the base address is still valid and pointing at the same game
  if (!vars.Watchers.GameId.Equals(vars.GameId)) {
    vars.BaseAddress = IntPtr.Zero;
    vars.Watchers = new MemoryWatcherList();
    return false;
  }
  
  // Add game-specific autosplitter logic here
  if (vars.Watchers["ByteValue"] == 1) {
    // ...
  }
  
  return true;
}


start {
   return (vars.Watchers["ByteValue"] == 1);
}

split {
   return (vars.Watchers["ByteValue"] == 1);
}

reset {
   return (vars.Watchers["ByteValue"] == 1);
}
