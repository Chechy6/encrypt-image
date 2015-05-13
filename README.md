
class Readfile{

public:

  int width, height, size;
  unsigned char* ReadBMP(char* filename);
  string strnum;
  unsigned short rgb_values[];

};


//extracting the data from BMP
unsigned char* Readfile::ReadBMP(char* filename)
{
FILE* f = fopen(filename, "rb");
if(f == NULL) throw "Argument Exception";

unsigned char info[54];

fread(info, sizeof(unsigned char), 54, f); // read the 54-byte header

// extract image height and width from header
width = *(int*)&info[18];
height = *(int*)&info[22];

cout << endl;
cout << "File name: " << filename << endl;
cout << "Width: " << width << endl;
cout << "Height: " << height << endl;

    size = 3 * width * height;
    unsigned short rgb_values[size];
    unsigned char* data = new unsigned char[size]; // allocate 3 bytes per pixel
    fread(data, sizeof(unsigned char), size, f); // read the rest of the data

//switch BGR to RGB
for(int i = 0; i < size; i += 3)
    {
            unsigned char tmp = data[i];
            data[i] = data[i+2];
            data[i+2] = tmp;

            rgb_values[i] = (short)data[i];
            rgb_values[i+1] = (short)data[i+1];
            rgb_values[i+2] = (short)data[i+2];

}

//put all RGB values into a single string strnum
std::ostringstream oss;

        for (int k=0; k < sizeof(rgb_values)/sizeof(rgb_values[0]); k ++)
            {
            oss << setfill('0') << setw(3) << rgb_values[k];
            }

    strnum = oss.str();
    fclose(f);

return data;
}


bool Writefile (unsigned char t[])
{

    FILE* r = fopen("D:\\CodelocksProject\\32x32.bmp", "rb");
    if(r == NULL) throw "Argument Exception";

unsigned char info[54];

fread(info, sizeof(unsigned char), 54, r); // read the 54-byte header

    FILE * w = fopen ("decrypted.bmp", "w");

       fwrite((int*)&info,sizeof(unsigned char),54,w);


unsigned char inf[54];

fread(inf, sizeof(unsigned char), 54, r); cout <<"FileHeader: ";
for (int i= 0; i<54; i++)
    {
        cout << *(int*)&inf[i]<<" ";
    }
fwrite(t,sizeof(unsigned char),sizeof(t)/sizeof(t[0]),w);
fclose(w);

return true;

}

//a function, returning "random" seed
    void get_rand(void* buf, int len)
    {
    char* bytes = (char*)buf;
    for (int i = 0; i != len; ++i) {
        bytes[i] = rand() & 0xff;
    }
    }


//Create a new file for the decrypted image
bool SaveImage(char* szPathName, unsigned char* lpBits, int w, int h)
{

FILE *pFile = fopen(szPathName, "wb");
if(pFile == NULL){return false;}

BITMAPINFOHEADER BMIH;
BMIH.biSize = sizeof(BITMAPINFOHEADER);
BMIH.biSizeImage = w * h * 3;

// Create the bitmap
BMIH.biSize = sizeof(BITMAPINFOHEADER);
BMIH.biWidth = w;
BMIH.biHeight = h;
BMIH.biPlanes = 1;
BMIH.biBitCount = 24;
BMIH.biCompression = BI_RGB;
BMIH.biSizeImage = w * h* 3;
BITMAPFILEHEADER bmfh;

int nBitsOffset = sizeof(BITMAPFILEHEADER) + BMIH.biSize;
LONG lImageSize = BMIH.biSizeImage;
LONG lFileSize = nBitsOffset + lImageSize;
bmfh.bfType = 'B'+('M'<<8);
bmfh.bfOffBits = nBitsOffset;
bmfh.bfSize = lFileSize;
bmfh.bfReserved1 = bmfh.bfReserved2 = 0;

//Write the bitmap file header
UINT nWrittenFileHeaderSize = fwrite(&bmfh, 1,
sizeof(BITMAPFILEHEADER), pFile);

//And then the bitmap info header
UINT nWrittenInfoHeaderSize = fwrite(&BMIH,
1, sizeof(BITMAPINFOHEADER), pFile);

//Finally, write the image data itself
UINT nWrittenDIBDataSize = fwrite(lpBits, 1, lImageSize, pFile);

fclose(pFile);

return true;

}





int main()
{
    //reading the .bmp file
    Readfile r;
    r.ReadBMP("D:/CodelocksProject/32x32.bmp");

    string text;
    text = r.strnum;
    FILE * pFile;
    pFile = fopen ("encrypted.bin", "wb");
    int N = text.length();
    int M = text.length()/128;
    string buffer;
    void* buf1;
    int at, pre=0, i;


    paillier_pubkey_t* pub;//The public key
    paillier_prvkey_t* prv;//The private key

        paillier_keygen(1024, &pub, &prv, get_rand);

      //encrypting the values with length 256 bytes
     for (i = 0; i < M; ++i)
        {
          at = (N+N*i)/M;

            buffer =  text.substr(pre, at-pre);

            paillier_plaintext_t* a = paillier_plaintext_from_str(buffer.c_str());
            paillier_ciphertext_t* ca = paillier_enc(0, pub, a, get_rand);

            buf1= paillier_ciphertext_to_bytes(PAILLIER_BITS_TO_BYTES(pub->bits)*2, ca);

            fwrite(buf1, 1, PAILLIER_BITS_TO_BYTES(pub->bits)*2, pFile);

            free(buf1);

            pre = at;
        }

        fclose (pFile);


cout << "Values length in bytes: " << buffer.size() << endl;


//Decryption

//reading the encrypted data from file
    FILE * qFile;
    long lSize;
    pFile = fopen ("encrypted.bin", "rb");
    fseek (pFile , 0 , SEEK_END);
    lSize = ftell (pFile);
    rewind (pFile);
    qFile = fopen ("example.bin", "wb");
    char * buff;
    void * buf2;
    size_t result;

    buff = (char*) malloc (sizeof(char)*lSize);
        if (buff == NULL) {fputs ("Memory error",stderr); exit (2);}
        result = fread (buff,1,lSize,pFile);



// separating the encrypted values, decrypting and writing them into .bin file
    for (pre = 0,i = 0; i < M; ++i)
        {
            at = (N+N*i)/M;

        void * f;
        f = (char*) &buff[pre*2];

        paillier_ciphertext_t* cb = paillier_ciphertext_from_bytes(f,PAILLIER_BITS_TO_BYTES(pub->bits)*2);
        paillier_plaintext_t* b = paillier_dec(0,pub,prv,cb);

          buf2 = paillier_plaintext_to_bytes(PAILLIER_BITS_TO_BYTES(pub->bits)*2,b);
          fwrite(buf2, 1, PAILLIER_BITS_TO_BYTES(pub->bits)*2, qFile);

            pre = at;
            free(buf2);
        }

fclose (qFile);

//put the decrypted data into one string
std::ifstream ifs("example.bin");
std::string str( (std::istreambuf_iterator<char>(ifs) ),(std::istreambuf_iterator<char>()));
ifs.close();


//removing white spaces
std::string str1 = str;
str1.erase(std::remove(str1.begin(), str1.end(), NULL), str1.end());


//converting the string into unsigned char array
          unsigned char usc[str1.size()/3];

        for (int k = 3; k <=(str1.size() - 3) ; k+=3)
            {
                ostringstream oss;
                oss << str1[k] << str1[k+1] << str1[k+2];
                istringstream iss(oss.str());
               short t;
               iss >> t;
                usc[k/3] = (unsigned char) t;

            }

//switching RGB to BGR for the new .bmp file
        for(int i = 0; i < str1.size()/3 ; i += 3)
            {
            unsigned char tmp = usc[i];
            usc[i] = usc[i+2];
            usc[i+2] = tmp;
            }

        cout << str1;
        SaveImage("decrypted.bmp",usc,r.width,r.height);

        //remove("example.bin");




    return 0;
}
