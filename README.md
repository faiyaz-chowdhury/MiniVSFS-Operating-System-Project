# MiniVSFS-Operating-System-Project

# MiniVSFS: A C-based VSFS Image Generator

This project implements a minimal version of the Very Simple File System (VSFS) called MiniVSFS. It consists of two main programs:

- `mkfs_builder`: Creates a raw disk image for the MiniVSFS file system
- `mkfs_adder`: Adds files to an existing MiniVSFS image

## Features

- **Block-based file system**: 4096-byte blocks
- **Inode-based**: 128-byte inodes with direct block pointers
- **Bitmap allocation**: Separate bitmaps for inodes and data blocks
- **Root directory support**: Single root directory (/) with . and .. entries
- **File addition**: Add files from the working directory to the file system
- **Checksum validation**: CRC32 checksums for data integrity

## File System Layout

```
[Superblock] [Inode Bitmap] [Data Bitmap] [Inode Table] [Data Region]
     1 block       1 block       1 block    N blocks      M blocks
```

## Building

### Using Make (Recommended)
```bash
make all
```

### Manual Compilation
```bash
gcc -Wall -Wextra -std=c99 -g -o mkfs_builder mkfs_builder.c
gcc -Wall -Wextra -std=c99 -g -o mkfs_adder mkfs_adder.c
```

## Usage

### mkfs_builder

Creates a new MiniVSFS file system image.

```bash
./mkfs_builder --image <output.img> --size-kib <180..4096> --inodes <128..512>
```

**Parameters:**
- `--image`: Name of the output image file
- `--size-kib`: Total size in kilobytes (must be multiple of 4, range: 180-4096)
- `--inodes`: Number of inodes in the file system (range: 128-512)

**Example:**
```bash
./mkfs_builder --image filesystem.img --size-kib 1024 --inodes 256
```

### mkfs_adder

Adds a file to an existing MiniVSFS image.

```bash
./mkfs_adder --input <input.img> --output <output.img> --file <filename>
```

**Parameters:**
- `--input`: Path to the input MiniVSFS image
- `--output`: Path for the updated output image
- `--file`: Name of the file to add (must exist in current directory)

**Example:**
```bash
echo "Hello World!" > hello.txt
./mkfs_adder --input filesystem.img --output filesystem_with_file.img --file hello.txt
```

## Testing

### Quick Test
```bash
make test
```

This will:
1. Create a test file system
2. Create a test file
3. Add the file to the file system
4. Display the file system structure

### Debug Information
```bash
make debug
```

This shows detailed information about the file system structure including:
- Superblock contents
- Bitmap states
- Root inode details

### Manual Testing

1. **Create a file system:**
   ```bash
   ./mkfs_builder --image test.img --size-kib 512 --inodes 128
   ```

2. **Create test files:**
   ```bash
   echo "This is a test file" > test1.txt
   echo "Another test file with more content" > test2.txt
   ```

3. **Add files to the file system:**
   ```bash
   ./mkfs_adder --input test.img --output test_v2.img --file test1.txt
   ./mkfs_adder --input test_v2.img --output test_v3.img --file test2.txt
   ```

4. **Examine the file system:**
   ```bash
   xxd test_v3.img | head -50
   ```

## File System Specifications

### Block Size
- **Size**: 4096 bytes
- **All structures**: Aligned to block boundaries

### Superblock (Block 0)
Contains file system metadata:
- Magic number: `0x4D565346` ("MVSF")
- Version, block size, total blocks
- Inode count and layout information
- Root inode number (always 1)
- Build timestamp

### Inodes
- **Size**: 128 bytes each
- **Direct blocks**: 12 pointers (no indirect blocks)
- **Maximum file size**: 48KB (12 × 4KB blocks)
- **Root inode**: Always at index 1

### Directory Entries
- **Size**: 64 bytes each
- **Name length**: Up to 57 characters + null terminator
- **Types**: File (1) or Directory (2)

### Bitmaps
- **Bit = 1**: Allocated
- **Bit = 0**: Free
- **Indexing**: Bit 0 refers to first object

## Error Handling

The programs handle various error conditions gracefully:

- **Invalid parameters**: Range checking for size and inode count
- **File not found**: Proper error messages for missing files
- **Insufficient space**: Detection of full file system
- **File too large**: Files exceeding 12 blocks are rejected
- **Duplicate files**: Prevention of adding files with same name
- **Corrupted file system**: Magic number validation

## Limitations

- **No indirect blocks**: Files limited to 12 direct blocks (48KB max)
- **Single directory**: Only root directory supported
- **No subdirectories**: Flat file system structure
- **No file deletion**: Files can only be added, not removed
- **No permissions**: Basic permission model only

## File System Integrity

All critical structures include checksums:
- **Superblock**: CRC32 checksum
- **Inodes**: CRC32 checksum
- **Directory entries**: 8-bit checksum

## Debugging Tools

### Hexdump Analysis
```bash
# View superblock
xxd -l 128 filesystem.img

# View inode bitmap
xxd -s 4096 -l 64 filesystem.img

# View data bitmap  
xxd -s 8192 -l 64 filesystem.img

# View root inode
xxd -s 12288 -l 128 filesystem.img
```

### Structure Offsets
- **Superblock**: Offset 0
- **Inode Bitmap**: Offset 4096 (Block 1)
- **Data Bitmap**: Offset 8192 (Block 2)  
- **Inode Table**: Offset 12288 (Block 3+)
- **Data Region**: Variable offset after inode table

## Project Structure

```
├── mkfs_builder.c    # Main builder program
├── mkfs_adder.c      # File addition program
├── Makefile          # Build configuration
└── README.md         # This documentation
```


## Clean Up

```bash
make clean
```

This removes all compiled binaries and test files.

## Troubleshooting

### Common Issues

1. **Compilation errors**: Ensure you have GCC with C99 support
2. **Permission denied**: Check file permissions for input files
3. **File too large**: Maximum file size is 48KB (12 blocks)
4. **Invalid image**: Verify the input image was created by mkfs_builder
5. **Insufficient space**: Check available inodes and data blocks

### Getting Help

Run programs with `--help` flag for usage information:
```bash
./mkfs_builder --help
./mkfs_adder --help
```
