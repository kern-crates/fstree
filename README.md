# fstree

File system management implementation

This module provides filesystem management functionality, including:

+ File and directory operations
+ Path manipulation and resolution
+ Permission and ownership control
+ Process filesystem context management

Core Components

+ FsStruct: Main filesystem context structure for a process
+ File and directory manipulation functions
+ Path resolution and canonicalization

## Examples

```rust
#![no_std]
#![no_main]

#[macro_use]
extern crate log;
extern crate alloc;

use alloc::string::ToString;
use core::panic::PanicInfo;

#[no_mangle]
pub extern "Rust" fn runtime_main(cpu_id: usize, dtb_pa: usize) {
    axlog2::init("debug");
    info!("[rt_fstree]: ...");

    fstree::init(cpu_id, dtb_pa);

    let fs = fstree::init_fs();
    let cwd = fs.lock().current_dir().unwrap_or("No CWD!".to_string());
    info!("cwd: {}", cwd);

    info!("[rt_fstree]: ok!");
    axhal::misc::terminate();
}

#[panic_handler]
pub fn panic(info: &PanicInfo) -> ! {
    arch_boot::panic(info)
}

```

## Structs

### `FsStruct`

```rust
pub struct FsStruct {
    pub users: i32,
    pub in_exec: bool,
    /* private fields */
}
```

Represents the filesystem context for a process

#### Implementations

**`impl FsStruct`**

```rust
pub fn init(&mut self, root_dir: Arc<RootDirectory>)
```

Initializes the filesystem context with a root directory

```rust
pub fn set_umask(&mut self, mode: u32)
```

Sets the file creation mask

```rust
pub fn copy_fs_struct(&mut self, fs: Arc<SpinLock<FsStruct>>)
```

Copies filesystem context from another process

```rust
pub fn lookup(
    &self,
    dir: Option<&VfsNodeRef>,
    path: &str,
    flags: i32
) -> AxResult<VfsNodeRef>
```

Looks up a file or directory in the filesystem

```rust
pub fn create_link(
    &self,
    dir: Option<&VfsNodeRef>,
    path: &str,
    node: VfsNodeRef
) -> AxResult
```

Creates a hard link to an existing file

```rust
pub fn create_symlink(
    &self,
    dir: Option<&VfsNodeRef>,
    path: &str,
    target: &str,
    uid: u32,
    gid: u32,
    mode: i32
) -> AxResult
```

Creates a symbolic link

```rust
pub fn create_file(
    &self,
    dir: Option<&VfsNodeRef>,
    path: &str,
    ty: VfsNodeType,
    uid: u32,
    gid: u32,
    mode: i32
) -> AxResult<VfsNodeRef>
```

Creates a new file or directory

```rust
pub fn create_dir(
    &self,
    dir: Option<&VfsNodeRef>,
    path: &str,
    uid: u32,
    gid: u32,
    mode: i32
) -> AxResult
```

Creates a new directory

```rust
pub fn root_dir(&self) -> Option<Arc<RootDirectory>>
```

Returns reference to the root directory

```rust
pub fn current_dir(&self) -> AxResult<String>
```

Returns reference to the current working directory

```rust
pub fn absolute_path(&self, path: &str) -> AxResult<String>
```

Returns absolute path from a possibly relative path

```rust
pub fn set_current_dir(&mut self, path: &str) -> AxResult
```

Changes current working directory

```rust
pub fn remove_file(&self, dir: Option<&VfsNodeRef>, path: &str) -> AxResult
```

Removes a file

```rust
pub fn remove_dir(&self, dir: Option<&VfsNodeRef>, path: &str) -> AxResult
```

Removes a directory

```rust
pub fn rename(&self, old: &str, new: &str) -> AxResult
```

Renames a file or directory

## Functions

### `init`

```rust
pub fn init(cpu_id: usize, dtb_pa: usize)
```

Initializes the filesystem subsystem

### `init_fs`

```rust
pub fn init_fs() -> Arc<SpinLock<FsStruct>>
```

Returns reference to the initialized filesystem context
