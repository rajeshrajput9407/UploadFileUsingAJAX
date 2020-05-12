# UploadFileUsingAJAX
In The MVC C#, we will upload file using AJAX.

     $('#FileUploadBtn').click(function () {
                debugger;
                // Checking whether FormData is available in browser
                if (window.FormData !== undefined) {

                    var fileUpload = $("#myFile").get(0);
                    var files = fileUpload.files;

                    // Create FormData object
                    var fileData = new FormData();

                    // Looping over all files and add it to FormData object
                    for (var i = 0; i < files.length; i++) {
                        fileData.append(files[i].name, files[i]);
                    }

                    // Adding one more key to FormData object
                    //fileData.append('username', ‘Manas’);

                    $.ajax({
                        url: '/Library/UploadFiles',
                        type: "POST",
                        contentType: false, // Not to set any content header
                        processData: false, // Not to process data
                        data: fileData,
                        success: function (result) {
                            if (result == "ok") {
                                alert("File Import Successfully");
                                window.location.reload();
                            } else {
                                alert(result);
                            }
                        },
                        error: function (err) {
                            alert(err.statusText);
                        }
                    });
                } else {
                    alert("FormData is not supported.");
                }
            });
            
            
            
            # MVC Action 
            [HttpPost]
        public ActionResult UploadFiles()
        {
            // Checking no of files injected in Request object  
            if (Request.Files.Count > 0)
            {
                try
                {
                    //  Get all files from Request object  
                    HttpFileCollectionBase files = Request.Files;
                    for (int i = 0; i < files.Count; i++)
                    {

                        HttpPostedFileBase file = files[i];
                        string fname;

                        if ((file != null) && (file.ContentLength != 0) && !string.IsNullOrEmpty(file.FileName))
                        {
                            string fileName = file.FileName;
                            string fileContentType = file.ContentType;
                            byte[] fileBytes = new byte[file.ContentLength];
                            var data = file.InputStream.Read(fileBytes, 0, Convert.ToInt32(file.ContentLength));


                            using (var package = new ExcelPackage(file.InputStream))
                            {
                                var currentSheet = package.Workbook.Worksheets;
                                var workSheet = currentSheet.First();
                                var noOfCol = workSheet.Dimension.End.Column;
                                var noOfRow = workSheet.Dimension.End.Row;

                                for (int rowIterator = 2; rowIterator <= noOfRow; rowIterator++)
                                {
                                    var AccessionNumber = _context.Tbl_LibraryBook.Count() + 1;
                                    var book = new Tbl_LibraryBooks();

                                    book.AccessionNo = Convert.ToString(AccessionNumber);
                                    book.BarCodeID = Convert.ToString(AccessionNumber + 1000);
                                    book.BarcodeImage = GenerateBarcode(AccessionNumber + 1000);

                                    book.Title = Convert.ToString(workSheet.Cells[rowIterator, 2].Value);
                                    book.Author = Convert.ToString(workSheet.Cells[rowIterator, 3].Value);
                                    book.Subject = Convert.ToString(workSheet.Cells[rowIterator, 4].Value);
                                    book.Publisher = Convert.ToString(workSheet.Cells[rowIterator, 5].Value);
                                    book.Edition = Convert.ToString(workSheet.Cells[rowIterator, 6].Value);
                                    book.Year = Convert.ToString(workSheet.Cells[rowIterator, 7].Value);
                                    book.Pages = Convert.ToString(workSheet.Cells[rowIterator, 8].Value);
                                    book.Vol = Convert.ToString(workSheet.Cells[rowIterator, 9].Value);
                                    book.Sourse = Convert.ToString(workSheet.Cells[rowIterator, 10].Value);
                                    book.BillNo = Convert.ToString(workSheet.Cells[rowIterator, 11].Value);
                                    book.Cost = Convert.ToString(workSheet.Cells[rowIterator, 12].Value);
                                    book.Course = Convert.ToString(workSheet.Cells[rowIterator, 13].Value);
                                    book.Semester = Convert.ToString(workSheet.Cells[rowIterator, 14].Value);
                                    book.BookNo = Convert.ToString(workSheet.Cells[rowIterator, 15].Value);
                                    book.WithdralNo = Convert.ToString(workSheet.Cells[rowIterator, 16].Value);
                                    book.Remark = Convert.ToString(workSheet.Cells[rowIterator, 17].Value);
                                    book.Location = Convert.ToString(workSheet.Cells[rowIterator, 18].Value);
                                    book.Topic = Convert.ToString(workSheet.Cells[rowIterator, 19].Value);
                                    book.ISBN = Convert.ToString(workSheet.Cells[rowIterator, 20].Value);
                                    book.Topic = Convert.ToString(workSheet.Cells[rowIterator, 21].Value);

                                    _context.Tbl_LibraryBook.Add(book);
                                    _context.SaveChanges();
                                }
                            }
                        }
                        fname = file.FileName;
                        // Get the complete folder path and store the file inside it.  
                        fname = Path.Combine(Server.MapPath("/Uploads/"), fname);
                        file.SaveAs(fname);
                    }
                    // Returns message that successfully uploaded  
                    return Json("ok");
                }
                catch (Exception ex)
                {
                    return Json("Error occurred. Error details: " + ex.Message);
                }
            }
            else
            {
                return Json("No files selected.");
            }
        }
