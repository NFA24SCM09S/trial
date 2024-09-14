// CS 525 GROUP 35 : Lavisha Chhatwani, Nidhi Niraj Shrivastav, Gladys Gince Skariah 

// Including necessary headers and define error codes for file operations, memory allocation, and error handling.
#include "storage_mgr.h"
#include "dberror.h"
#include "test_helper.h"
#include "stdio.h"
#include "stdlib.h"
#include "string.h"
#include "unistd.h"
#define RC_MEMORY_ALLOCATION_FAILED 
#define RC_READ_FAILED 


//Declaring a common file pointer to be used across the functions
FILE *fileptr;


// Creating Variable to track status codes for different operations
int checkStatus = 0;


// Initialize the storage manager. This function does for the storage manager.
 
extern void initStorageManager(void)
{
    fileptr = NULL; //sets the file pointer(fileptr) to NULL.
    printf("Storage Manager Initialized\n");
}


// Create a new page file. File is set to a size of 1 page, filled with '\0'. 
 
extern RC createPageFile(char *fileName)
{
    // Open the file in write mode using built in fopen function (file will be created if it does not exists).
    fileptr = fopen(fileName, "w");
    if (fileptr == NULL)
    {
        return RC_FILE_NOT_FOUND; // Return error if the file could not be created
    }
    
    
    // Allocate memory for 1 page and initialize it to 0
    char *mem = (char *)calloc(PAGE_SIZE, sizeof(char));
    if (mem == NULL)
    {
        fclose(fileptr);  // Close file before returning if memory allocation fails
        return RC_MEMORY_ALLOCATION_FAILED;
    }
    

    // Write the page to the file using built in fwrite function
    size_t totalWritten = fwrite(mem, sizeof(char), PAGE_SIZE, fileptr);
    
    // Check to see if the written elements is within the given page limit
    if (totalWritten < PAGE_SIZE)
    {
    	free(mem);   
        fclose(fileptr); 
        return RC_WRITE_FAILED;
    }
    
    // Clean up memory and close the file
    fclose(fileptr);
    free(mem);
    return RC_OK;
}


// Opening an existing page file. 
   
extern RC openPageFile(char *fileName, SM_FileHandle *fHandle)
{
    // Opens file in read mode.
    fileptr = fopen(fileName, "r");
    if (fileptr == NULL)
    {
        return RC_FILE_NOT_FOUND; // Will Return an error incase file could not be opened
    }
    
    // Move the file pointer to the end to calculate the file size.
    if (fseek(fileptr, 0, SEEK_END) != 0)
    {
        fclose(fileptr);
        return RC_FILE_SEEK_FAILED;
    }

    int size = ftell(fileptr);
    if (size == -1)
    {
        fclose(fileptr);
        return RC_READ_NON_EXISTING_PAGE;
    }

    // Populate the file handle with metadata
    fHandle->curPagePos = 0;
    fHandle->fileName = fileName;
    fHandle->totalNumPages = size / PAGE_SIZE;

    fclose(fileptr);  // Close the file after gathering metadata
    return RC_OK;
}

// Closing a page file. This function opens the file in read mode, then immediately closes it. 

extern RC closePageFile(SM_FileHandle *fHandle)
{
    fileptr = fopen(fHandle->fileName, "r"); // Open the file in read mode.
    if (fileptr == NULL)
    {
        return RC_FILE_NOT_FOUND; // Return an error if the file could not be opened.
    }

    // Closing file after opening it.
    fclose(fileptr);
    return RC_OK;
}

// Destroying a page file. This operation permanently removes the file. 

extern RC destroyPageFile(char *fileName)
{
    if (remove(fileName) == 0) // Attempt to remove the file.
    {
        return RC_OK;  // Successfully removed the file.
    }
    return RC_FILE_NOT_FOUND;  // File could not be found so returns an error.
}


// Reading a specific block from the page file. The block is read into the provided memory page. 

extern RC fetchBlock(int pageNum, SM_FileHandle *fHandle, SM_PageHandle memPage)
{
    fileptr = fopen(fHandle->fileName, "r");
    if (fileptr == NULL)
    {
        return RC_FILE_NOT_FOUND; // Return an error if the file could not be opened.
    }

    if (pageNum >= fHandle->totalNumPages || pageNum < 0)  // Check if the requested page number is within the valid range.
    {
        fclose(fileptr);
        return RC_READ_NON_EXISTING_PAGE;
    }

    // Move file pointer to the position of the requested page.
    if (fseek(fileptr, pageNum * PAGE_SIZE, SEEK_SET) != 0)
    {
        fclose(fileptr);
        return RC_FILE_SEEK_FAILED;
    }

    // Read the page into memory.
    size_t bytesRead = fread(memPage, sizeof(char), PAGE_SIZE, fileptr);
    if (bytesRead < PAGE_SIZE)
    {
        fclose(fileptr);
        return RC_READ_FAILED;
    }

    // Update the current page position in the file handle
    fHandle->curPagePos = pageNum;

    // Close the file after reading.
    fclose(fileptr);
    return RC_OK;
}

//  Get the current block position in the page file.
 
extern int getBlockPos(SM_FileHandle *fHandle)
{
    if (fHandle == NULL)
    {
        return RC_FILE_HANDLE_NOT_INIT; // Return an error if the file handle is not initialized.
    }
    return fHandle->curPagePos; // Return the current page position.
}

// Read the first block in the file. This function utilizes fetchBlock() with page number 0.

extern RC readFirstBlock(SM_FileHandle *fHandle, SM_PageHandle memPage)
{
    return fetchBlock(0, fHandle, memPage);
}

//  Read the previous block from the current position. It will calculate previous page number and will read it.
 
extern RC readPreviousBlock(SM_FileHandle *fHandle, SM_PageHandle memPage)
{
    int prevPage = fHandle->curPagePos - 1;
    if (prevPage < 0)
    {
        return RC_READ_NON_EXISTING_PAGE;
    }
    return fetchBlock(prevPage, fHandle, memPage);
}

// Reading the current block in the file. This function calls fetchBlock() with the current page position.
 
extern RC readCurrentBlock(SM_FileHandle *fHandle, SM_PageHandle memPage)
{
    return fetchBlock(fHandle->curPagePos, fHandle, memPage);
}

// Reading the next block from the current position while ensuring the next page exists before attempting to read.
 
extern RC readNextBlock(SM_FileHandle *fHandle, SM_PageHandle memPage)
{
    int nextPage = fHandle->curPagePos + 1;
    if (nextPage >= fHandle->totalNumPages)
    {
        return RC_READ_NON_EXISTING_PAGE;
    }
    return fetchBlock(nextPage, fHandle, memPage);
}

// Reading the last block in the file. This function calls fetchBlock() with the last page number.
 
extern RC readLastBlock(SM_FileHandle *fHandle, SM_PageHandle memPage)
{
    return fetchBlock(fHandle->totalNumPages - 1, fHandle, memPage);
}

// Writing a block to the page file. The block is written at the specified page number. */

extern RC storeBlock(int pageNum, SM_FileHandle *fHandle, SM_PageHandle memPage)
{
    fileptr = fopen(fHandle->fileName, "r+"); //  Opens the file in read-write mode.
    if (fileptr == NULL)
    {
        return RC_FILE_NOT_FOUND; // Returns an error if the file could not be opened.
    }

    if (pageNum > fHandle->totalNumPages || pageNum < 0) // Checks if the provided page number is valid
    {
        fclose(fileptr); // Close the file before returning an error.
        return RC_WRITE_FAILED;
    }

    if (fseek(fileptr, pageNum * PAGE_SIZE, SEEK_SET) != 0)// Moves fileptr to position corresponding to specified page no.
    {
        fclose(fileptr); // Close the file before returning an error if seeking fails.
        return RC_FILE_SEEK_FAILED;
    }

    size_t bytesWritten = fwrite(memPage, sizeof(char), PAGE_SIZE, fileptr); // Write the contents of memPage to the file at the current position
    if (bytesWritten < PAGE_SIZE)
    {
        fclose(fileptr); // Close the file before returning an error if the write operation fails.
        return RC_WRITE_FAILED;
    }

    fHandle->curPagePos = pageNum; // Update the file handle to reflect the current position after the write operation.
    fclose(fileptr); // Close the file and return success.
    return RC_OK;
}

// Write to the current block in the page file. This function calls storeBlock() with the current page position.

extern RC writeCurrentBlock(SM_FileHandle *fHandle, SM_PageHandle memPage)
{
    return storeBlock(fHandle->curPagePos, fHandle, memPage); // Will call storeBlock with recent page position.
}


// Appends a empty block at extreme end of the page file. This function adds a new block of zeroed bytes and updates the total page count. 
extern RC appendEmptyBlock(SM_FileHandle *fHandle)
{
	// Opens in append mode.
    fileptr = fopen(fHandle->fileName, "a+");
    if (fileptr == NULL)
    {
        return RC_FILE_NOT_FOUND;  // Return an error if the file could not be opened.
    }

    // Allocate memory for a new block of size PAGE_SIZE and initialize it with zeros.
    SM_PageHandle emptyBlock = (SM_PageHandle)calloc(PAGE_SIZE, sizeof(char));
    if (emptyBlock == NULL)
    {
        fclose(fileptr); // Close the file and return an error if memory allocation fails.
        return RC_MEMORY_ALLOCATION_FAILED;
    }

    // Write the empty block to the end of the file.
    size_t bytesWritten = fwrite(emptyBlock, sizeof(char), PAGE_SIZE, fileptr);
    if (bytesWritten < PAGE_SIZE)
    {
        free(emptyBlock); // Frees allocated memory and close the file before returning an error if the write operation fails.
        fclose(fileptr);
        return RC_WRITE_FAILED;
    }

    fHandle->totalNumPages++; // Update the total number of pages in the file handle.
    
    fclose(fileptr);
    free(emptyBlock);
    return RC_OK;
}

// Checks if the file has atleast specified no. of pages. If not, empty pages are appended until the file reaches the desired size.

extern RC ensureCapacity(int numberOfPages, SM_FileHandle *fHandle)
{
    if (fHandle->totalNumPages >= numberOfPages) // Check if the file already has the required number of pages.
    {
        return RC_OK; // Return success if the file has enough pages.
    }
    
    // Append empty blocks until the file reaches the required number of pages.
    while (fHandle->totalNumPages < numberOfPages)
    {
        appendEmptyBlock(fHandle);
    }

    return RC_OK;
}

// Utility function to check operation status. Updates status based on the result of previous operations.
extern void check_Status()
{
	// If the previous operation was successful (RC_OK), update the status to 1.
    if (checkStatus == RC_OK)
    {
        checkStatus = 1;
    }
}
