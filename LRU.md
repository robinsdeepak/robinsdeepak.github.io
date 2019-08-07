## LRU Cache
LRU cache stands for Least Recently Used cache, it is a cache replacement algorithm, as the name says it evicts the least recently used memory in order to provide fast and efficient way of retrieving data.

Below code is the implementation of LRU cache in c language, I've skipped the commenting of this script rather i've explained every functions and blocks in detail.



**queue_node**: It's a structure defined for a node of the doubly linked list queue
                it holds the attributes: prev_node -- pointer to previous node
                next_node -- pointer to next_node
                page_number;
                
    struct queue_node {
      struct queue_node *prev_node, *next_node;
      int page_number;  
    };


**structue Queue** will be used to keep the information about the Queue
                   attributes: number of filled frames & total number of frames

    struct Queue {
      int number_of_filled_frames;
      int number_of_frames;
      struct queue_node *front, *rear;
    };


structure **Hash** is the Collection of addresses of Queue nodes  

    struct Hash {
      int capacity;
      struct queue_node** array;
    };

**create_queue_node:** function defined to create a Queue node on demand
  parameters: page_number to be stored
  return: pointer to the created node

    struct queue_node* create_queue_node(int page_number){
      struct queue_node* temp = (struct queue_node*)malloc(sizeof(struct queue_node));
      temp->page_number = page_number;

      temp->prev_node = temp->next_node = NULL;

      return temp;
    }

**create_queue:** Function to create a queue with given number of frames

    struct Queue* create_queue(int number_of_frames){
        struct Queue* queue = (struct Queue*)malloc(sizeof(struct Queue));

        queue->number_of_filled_frames = 0;
        queue->front = queue->rear = NULL;

        queue->number_of_frames = number_of_frames;

        return queue;
    }

**create_hash:** function create_hash will be used to create a empty hash of given capacity

    struct Hash* create_hash(int capacity){
        struct Hash* hash = (struct Hash*)malloc(sizeof(struct Hash));
        hash->capacity = capacity;

        hash->array = (struct queue_node**)malloc(hash->capacity * sizeof(struct queue_node*));

        int i;
        for (i = 0; i < hash->capacity; ++i)
            hash->array[i] = NULL;

        return hash;
    }
    
**check_frames_full**: this funtion will return the status of Queue if it is full or not

    int check_frames_full(struct Queue* queue){
        return queue->number_of_filled_frames == queue->number_of_frames;
    }

**check_empty_queue: ** this funtion takes a pointer to queue and returns a boolean 1 or 0 based on if queue is empty or not

    int check_empty_queue(struct Queue* queue){
        return queue->rear == NULL;
    }

**delete_queue**: a function to delete a frame from a queue

    void delete_queue(struct Queue* queue){
        if (check_empty_queue(queue))
            return;

        if (queue->front == queue->rear)
            queue->front = NULL;

        struct queue_node* temp = queue->rear;
        queue->rear = queue->rear->prev_node;

        if (queue->rear)
            queue->rear->next_node = NULL;

        free(temp);

        queue->number_of_filled_frames--;
    }

**add_a_page**: a funtion to add a page to queue and has with given page_number

    void add_a_page(struct Queue* queue, struct Hash* hash, int page_number){
        if (check_frames_full(queue)) {
            hash->array[queue->rear->page_number] = NULL;
            deQueue(queue);
        }

        struct queue_node* temp = create_queue_node(page_number);
        temp->next_node = queue->front;

        if (check_empty_queue(queue))
            queue->rear = queue->front = temp;
        else{
            queue->front->prev_node = temp;
            queue->front = temp;
        }

        hash->array[page_number] = temp;

        queue->number_of_filled_frames++;
    }



**reference_a_page**: This function will allow us to reference a page in a Queue and Hash

    void reference_a_page(struct Queue* queue, struct Hash* hash, int page_number){
        struct queue_node* requested_page = hash->array[page_number];

        if (requested_page == NULL)
            add_a_page(queue, hash, page_number);

        else if (requested_page != queue->front) {
            requested_page->prev_node->next_node = requested_page->next_node;
            if (requested_page->next_node)
                requested_page->next_node->prev_node = requested_page->prev_node;

            if (requested_page == queue->rear) {
                queue->rear = requested_page->prev_node;
                queue->rear->next_node = NULL;
            }

            requested_page->next_node = queue->front;
            requested_page->prev_node = NULL;

            requested_page->next_node->prev_node = requested_page;

            queue->front = requested_page;
        }
    }

### main() function: 
  Inside this function we create a Queue, Hash and finally we will perform multiple sequencial reference to pages

    int main(){
        struct Queue* queue = create_queue(4);
        struct Hash* hash = create_hash(10);

        reference_a_page(queue, hash, 1);
        reference_a_page(queue, hash, 2);
        reference_a_page(queue, hash, 3);
        reference_a_page(queue, hash, 1);
        reference_a_page(queue, hash, 4);
        reference_a_page(queue, hash, 5);
        reference_a_page(queue, hash, 6);

        printf("%d ", queue->front->page_number);
        printf("%d ", queue->front->next_node->page_number);
        printf("%d ", queue->front->next_node->next_node->page_number);
        printf("%d ", queue->front->next_node->next_node->next_node->page_number);

        return 0;
    }
