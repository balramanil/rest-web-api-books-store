AUTHORS.CS (CODE BEHIND)

namespace BooksAPI.Models
{
    public class Author
    {
        public int AuthorId { get; set; }
        public string AuthorName { get; set; }
    }
}




BOOKSS.CS (CODE BEHIND)

namespace BooksAPI.Models
{
    public class Book
    {
        public int BookId { get; set; }
        public string Title { get; set; }
        public bigint ISBN { get; set; }
        public string Language { get; set; }
        public string Description { get; set; }
        public int AuthorId { get; set; }
        public Author Author { get; set; }
    }
}





protected override void Seed(BooksAPI.Models.BooksAPIContext context)
{
    context.Authors.AddOrUpdate(new Author[] {
        new Author() { AuthorId = 1, AuthorName = "cale, john" },
        new Author() { AuthorId = 2, AuthorName = "Geithner, Timothy" },
        new Author() { AuthorId = 3, AuthorName = "Miller, Davis" },
        new Author() { AuthorId = 4, AuthorName = "WEST, ADAM" },
        new Author() { AuthorId = 5, AuthorName = "LEE, CHRISTOPHER" }
        });

        context.Books.AddOrUpdate(new Book[] {
        new Book() { BookId = 1,  Title= "WHAT'S WELSH FOR ZEN", ISBN = 0747543836, 
         Language = "English", AuthorId = 1, AuthorName = "cale, john", Description = "The Autobiography of JOHN CALE."}, 

         new Book() { BookId = 2,  Title= "STRESS TEST", ISBN = 0804138591, 
         Language = "English", AuthorId = 2, AuthorName = "Geithner, Timothy", Description = "Reflections on Financial Crises."}, 

         new Book() { BookId = 3,  Title= "The Tao of Muhammad Ali", ISBN = 0446519464, 
         Language = "English", AuthorId = 3, Description ="The front free end paper is inscribed, To Mal love Davis."}, 

         new Book() { BookId = 4,  Title= "BACK to the BATCAVE ", ISBN = 0425143708, 
         Language = "French", AuthorId = 4, AuthorName = "WEST, ADAM", Description = "The star of television's legendary show Batman gives a behind-the-scenes."},

         new Book() { BookId = 5,  Title= "TALL, DARK and GRUESOME", ISBN = 1887664254, 
         Language = "English", AuthorId = 5, AuthorName = "LEE, CHRISTOPHER", Description =
        "Christopher Lee, international film actor, famed for his work in the horror genre, details his childhood, war years, 
        friendships with Peter Cushing, Vincent Price, Robert Bloch and Boris Karloff, and, of course, his varied and interesting film career."}, 
    });
}




GET request to /api/books/1

{
  "BookId": 1,
  "Title": "WHAT'S WELSH FOR ZEN",
  "ISBN": 0747543836,
  "Language": "English",
  "Description": "The Autobiography of JOHN CALE.",
  "AuthorId": 1,
  "AuthorName" = "cale, john"
}

GET request to /api/books/2

{
  "BookId": 2,
  "Title": "STRESS TEST",
  "ISBN": 0804138591,
  "Language": "English",
  "Description": "Reflections on Financial Crises.",
  "AuthorId": 2,
  "AuthorName" = "Geithner, Timothy"
}

GET request to /api/books/3

{
  "BookId": 3,
  "Title": "The Tao of Muhammad Ali",
  "ISBN": 0446519464,
  "Language": "English",
  "Description": "The front free end paper is inscribed, To Mal love Davis.",
  "AuthorId": 3,
  "AuthorName":null
}

GET request to /api/books/4

{
  "BookId": 4,
  "Title": "BACK to the BATCAVE",
  "ISBN": 0425143708,
  "Language": "French",
  "Description": "The star of television's legendary show Batman gives a behind-the-scenes.",
  "AuthorId": 4,
  AuthorName = "WEST, ADAM"
}

GET request to /api/books/5

{
  "BookId": 5,
  "Title": "TALL, DARK and GRUESOME",
  "ISBN": 1887664254,
  "Language": "English",
  "Description": "Christopher Lee, international film actor, famed for his work in the horror genre, details his childhood, war years, 
       friendships with Peter Cushing, Vincent Price, Robert Bloch and Boris Karloff, and, of course, his varied and interesting film career.",
  "AuthorId": 5,
  AuthorName = "LEE, CHRISTOPHER"
}




namespace BooksAPI.DTOs
{
    public class BookDto
    {
        public string Title { get; set; }
        public string Author { get; set; }
        public string Language { get; set; }
    }
}




using System;

namespace BooksAPI.DTOs
{
    public class BookDetailDto
    {
        public string Title { get; set; }
        public string Language { get; set; }
        public Bigint ISBN { get; set; }
        public string Description { get; set; }        
        public string Author { get; set; }
    }
}



Controller Code

namespace BooksAPI.Controllers
{
    [RoutePrefix("api/books")]
    public class BooksController : ApiController
    {
        private BooksAPIContext db = new BooksAPIContext();

        // Typed lambda expression for Select() method. 
        private static readonly Expression<Func<Book, BookDto>> AsBookDto =
            x => new BookDto
            {
                Title = x.Title,
                Author = x.Author.AuthorName,
                ISBN = x.ISBN
            };

 
        [Route("")]
        public IQueryable<BookDto> GetBooks()
        {
            return db.Books.Include(b => b.Author).Select(AsBookDto);
        }
        
// GET api/Books
    {"Title":"WHAT'S WELSH FOR ZEN","Author":"cale, john","ISBN":0747543836}
    {"Title":"STRESS TEST","Author":"Geithner, Timothy","ISBN":0804138591}
    {"Title":"The Tao of Muhammad Ali","Author":null,"ISBN":0446519464}
    {"Title":"BACK to the BATCAVE","Author":"WEST, ADAM","ISBN":0425143708}
    {"Title":"TALL, DARK and GRUESOME","Author":"LEE, CHRISTOPHER","ISBN":1887664254}





        
        [Route("{id:int}")]
        [ResponseType(typeof(BookDto))]
        public async Task<IHttpActionResult> GetBook(int id)
        {
            BookDto book = await db.Books.Include(b => b.Author)
                .Where(b => b.BookId == id)
                .Select(AsBookDto)
                .FirstOrDefaultAsync();
            if (book == null)
            {
                return NotFound();
            }

            return Ok(book);
        }
        
// GET api/Books/2
        {"Title":"STRESS TEST","Author":"Geithner, Timothy","ISBN":0804138591}





        [Route("{id:int}/details")]
        [ResponseType(typeof(BookDetailDto))]
        public async Task<IHttpActionResult> GetBookDetail(int id)
        {
            var book = await (from b in db.Books.Include(b => b.Author)
                              where b.AuthorId == id
                              select new BookDetailDto
                              {
                                  Title = b.Title,
                                  ISBN = b.ISBN,
                                  Language = b.Language,
                                  Description = b.Description,
                                  Author = b.Author.AuthorName
                              }).FirstOrDefaultAsync();

            if (book == null)
            {
                return NotFound();
            }
            return Ok(book);
        }
              
// GET api/books/2/details    
        {
          "Title": "STRESS TEST",
          "ISBN": "0804138591",
          "Language": "English",
          "Description": "Reflections on Financial Crises.",
          "AuthorName" = "Geithner, Timothy"
        }

        
        
       

        [Route("{ISBN:Bigint}")]
        public IQueryable<BookDto> GetBooksByGenre(Bigint ISBN)
        {
            return db.Books.Include(b => b.Author)
                .Where(b => b.ISBN.Equals(ISBN, StringComparison.OrdinalIgnoreCase))
                .Select(AsBookDto);
        }
        
// GET api/books/0446519464
        {
      "Title": "The Tao of Muhammad Ali",
      "ISBN": "0446519464",
      "Language": "English",
      "Description": "The front free end paper is inscribed, To Mal love Davis.",
      "AuthorName":null
        }

        
        
        

        [Route("~api/authors/{authorId}/books")]
        public IQueryable<BookDto> GetBooksByAuthor(int authorId)
        {
            return db.Books.Include(b => b.Author)
                .Where(b => b.AuthorId == authorId)
                .Select(AsBookDto);
        }
        
// GET api/books/2   
        {
          "Title": "STRESS TEST",
          "ISBN": "0804138591",
          "Language": "English",
          "Description": "Reflections on Financial Crises.",
          "AuthorName" = "Geithner, Timothy"
        }
        
        
        
        


        [Route("{language}")]
        public IQueryable<BookDto> GetBooksByGenre(string language)
        {
            return db.Books.Include(b => b.Author)
                .Where(b => b.Language.Equals(language, StringComparison.OrdinalIgnoreCase))
                .Select(AsBookDto);
        }

// Get api/books/French
      [{"Title": "BACK to the BATCAVE", "ISBN": 0425143708, AuthorName = "WEST, ADAM",
      "Description": "The star of television's legendary show Batman gives a behind-the-scenes.", "Language": "French"}]    
        
        
        
       

        protected override void Dispose(bool disposing)
        {
            db.Dispose();
            base.Dispose(disposing);
        }
    }
}
