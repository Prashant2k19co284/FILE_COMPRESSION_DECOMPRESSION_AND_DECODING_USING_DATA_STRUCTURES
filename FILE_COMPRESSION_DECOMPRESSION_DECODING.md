# 1)  HUFFMAN CODING IS A COMPRESION ALGORITHM THOSE STRING HAVING MORE FREQUENCY WILL TAKE LESS SPACE AND HAVING DIFFERENT CODE FOR EACH VARIABLE STRING AND THOSE HAVING LESS FREQUENCY WILL TAKE MORE SPACE (MEANS FILE IS COMPRESSED TO ZIP FILE )

MAKING CODE FOR A CHARACTER BASED ON THEIR FREQUENCY AND REDUCES THE SPACE IN THE MEMORY 

DATA STRUCTURE USED FOR COMPRESSING THE STRING  1) HASH MAP FOR FREQUENCY DICTIONARY 
                                                2) TREE
                                                3) MIN HEAP 
                                                4) HASH MAP FOR CHARACTERS AND BITS 


DATA STRUCTURE USED FOR DECODING THE STRING     1) REVERSE HASP MAP OF CAHRACTERS AND BITS (PREFIX FREE BITS CODE BEACUSE ALL                                                      NODES ARE LEAF NODES ) 
                                                2) ASSIGN THE CORRESPONDING VALUE TO IT 


```python
import heapq
import os


class BinaryTreeNode:
    def __init__(self,value,freq):
        self.value=value
        self.freq=freq
        self.right=None
        self.left=None
        
        
    def __lt__(self,other):
        if other is not None:
            return self.freq<other.freq
    
    
    def __eq__(self,other):
        if other is not None:
            return self.freq==other.freq
    
    
class Huffman_coding:
    def __init__(self,path):
        self.path=path
        self.__heap=[]
        self.__codes={}
        self.__reverse_codes={}

    
    
    
    #MAKING THE FREQUENCY DICTIONARY FOR EACH CHARACTER WITH HELPER FUNCTION 
    def __make_frequency_dictionary(self,text):
        freq_dict={}
        for char in text:
            if char not in freq_dict:
                freq_dict[char]=0
            freq_dict[char]+=1
        return freq_dict
    
    
    
    
    #MAKING THE BUILD HEAP WITH THE HEALPER FUNCTION
    def __buildheap(self,freq_dict):
        for key in freq_dict:
            frequency=freq_dict[key]
            binary_tree_node=BinaryTreeNode(key,frequency)
            heapq.heappush(self.__heap,binary_tree_node)
    
    
    
    #MAKING THE BINARY TREE WITH THE HELPER FUNCTION
    def __buildtree(self):
        while len(self.__heap)>1:
            binary_tree_node_1=heapq.heappop(self.__heap)
            binary_tree_node_2=heapq.heappop(self.__heap)
            freq_sum=binary_tree_node_1.freq+binary_tree_node_2.freq
            newnode=BinaryTreeNode(None,freq_sum)
            newnode.left=binary_tree_node_1
            newnode.right=binary_tree_node_2
            heapq.heappush(self.__heap,newnode)
        return 
    
    
    
    #FUNTION HELPER TO BUILD THE CODES CORROSPONDING TO THE GIVEN CHAR
    def __buildcodeshelper(self,root,curr_bits):
        if root==None:
            return 
        if root.value is not None:
            self.__codes[root.value]=curr_bits
            self.__reverse_codes[curr_bits]=root.value
            return 
        self.__buildcodeshelper(root.left,curr_bits+"0")
        self.__buildcodeshelper(root.right,curr_bits+"1")
    
    
    
    #HELPER FUNCTION TO BUILDCODES
    def __buildcodes(self):
        root=heapq.heappop(self.__heap)
        self.__buildcodeshelper(root,"")
    
    
    
    #FUNTION USED TO GET ENCODED TEXT
    def __getEncoded_text(self,text):
        Encodedtext=""
        for char in text:
            Encodedtext+=self.__codes[char]
        return Encodedtext
    
    #PADDED THE ENCODED TEXT 
    def __getpadded_text(self,Encoded_text):
        padded_amount=8-(len(Encoded_text)%8)
        
        for i in range(padded_amount):
            Encoded_text+="0"
        padded_info="{0:08b}".format(padded_amount)
        Padded_encoded_text=padded_info+Encoded_text
        return Padded_encoded_text
    
    
    #CONVERTING THE PADDED TEXT INTO BYTES ARRAY 
    def __getBytesArray(self,Padded_encoded):
        array=[]
        for i in range(0,len(Padded_encoded),8):
            byte=Padded_encoded[i:i+8]
            array.append(int(byte,2))
        return array
    
    
    def compress(self):
            
        #GET FILE FROM PATH
        file_name,file_extension=os.path.splitext(self.path)
        output_path=file_name+".bin"
        
        with open(self.path,'r+') as file ,open(output_path,'wb') as output:
            
            #READ TEXT FROM FILE 
            text=file.read()
            text=text.rstrip()
            
            #MAKE FREQUENCY DICTIONARY USING HASH MAP 
            freq_dict=self.__make_frequency_dictionary(text)
            
            
            #CONSTRUCT THE HEAP USING FREQUENCY DICTIONARY
            self.__buildheap(freq_dict)
            
            
            #CONSTRUCT THE BINARY TREE FROM THE HEAP 
            self.__buildtree()
            
            
            #CONSTRUCT THE CODE FROM BINARY TREE
            self.__buildcodes()
            
            
            #CREATING THE ENCODED TEXT USING THE CODES 
            Encodedtext=self.__getEncoded_text(text)
            
            
            #PADDING OF THE ENCODED TEXT 
            Padded_encoded=self.__getpadded_text(Encodedtext)
            
            
            #MAKING BYTES_ARRAY WITH THE PADDED TEXT 
            Bytes_array=self.__getBytesArray(Padded_encoded)
            
            
            #RETURN THE BINARY FILE AS OUTPUT
            Final_Bytes=bytes(Bytes_array)
            output.write(Final_Bytes)
            print("Compressed")
            return output_path
        
    
    # REMOVING THE PADDING TO GET ACTUALL TEXT
    def __removePadding(self,text):
        padded_info=text[:8]
        extra_padding=int(padded_info,2)
        text=text[8:]
        text_after_remove_padding=text[:-1*extra_padding]
        return text_after_remove_padding
    
    
    #DECOMPRESSED TEXT 
    def __decode_text(self,text):
        decoded_text=""
        current_bits=""
        
        for bit in text:
            current_bits+=bit
            if current_bits in self.__reverse_codes:
                character=self.__reverse_codes[current_bits]
                decoded_text+=character
                current_bits=""
            
        return decoded_text
        
    
    
    #DECOMPRESSION OF BITS TO BINARY BITS STRING 
    def decompress(self,input_path):
        filename,file_extension=os.path.splitext(self.path)
        output_path=filename+"_decompressed"+".txt"
        with open(input_path,'rb') as file,open(output_path,'w') as output:
            bit_string=""
            byte=file.read(1)
            while byte:
                byte=ord(byte)
                bits=bin(byte)[2:].rjust(8,'0')
                bit_string+=bits
                byte=file.read(1)
            actual_text=self.__removePadding(bit_string)
            decompressed_text=self.__decode_text(actual_text)
            output.write(decompressed_text)
            
        return 
                
    
            
```


```python
path=r"C:\Users\HP\Desktop\huffman.txt"
h=Huffman_coding(path)
output_path=h.compress()
h.decompress(output_path)
```

    Compressed
    


```python

```
