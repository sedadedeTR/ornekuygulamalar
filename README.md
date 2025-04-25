# ornekuygulamalar
Eğitim ve Demolarda kullanilan ornek uygulamalar
using System;
using System.Runtime.InteropServices;

class Program
{
    const int MAX_PREFERRED_LENGTH = -1;
    const int NERR_Success = 0;

    [StructLayout(LayoutKind.Sequential, CharSet = CharSet.Unicode)]
    struct SHARE_INFO_1
    {
        public string shi1_netname;
        public uint shi1_type;
        public string shi1_remark;
    }

    [DllImport("Netapi32.dll", CharSet = CharSet.Unicode)]
    static extern int NetShareEnum(
        string servername,
        int level,
        out IntPtr bufptr,
        int prefmaxlen,
        out int entriesread,
        out int totalentries,
        ref int resume_handle);

    [DllImport("Netapi32.dll")]
    static extern int NetApiBufferFree(IntPtr Buffer);

    static void Main()
    {
        string server = "110.60.193.49"; // ya da "an850"
        IntPtr buffer = IntPtr.Zero;
        int entriesRead = 0, totalEntries = 0, resume = 0;

        int result = NetShareEnum(server, 1, out buffer, MAX_PREFERRED_LENGTH,
            out entriesRead, out totalEntries, ref resume);

        if (result == NERR_Success)
        {
            int structSize = Marshal.SizeOf(typeof(SHARE_INFO_1));

            for (int i = 0; i < entriesRead; i++)
            {
                IntPtr item = IntPtr.Add(buffer, i * structSize);
                SHARE_INFO_1 share = Marshal.PtrToStructure<SHARE_INFO_1>(item);

                // 0x2 = yazıcı paylaşımı
                if (share.shi1_type == 0x2)
                {
                    Console.WriteLine($"\\\\{server}\\{share.shi1_netname}");
                }
            }

            NetApiBufferFree(buffer);
        }
        else
        {
            Console.WriteLine($"NetShareEnum başarısız. Hata kodu: {result}");
        }
    }
}
